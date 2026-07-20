# V20 Implementation Plan: Fold-Safe Target Encoding

**Status: historical.** This spec was fully implemented and reviewed; see
"V20 Fold-Safe Target Encoding Review" in
`docs/9_leaderboard_improvement_insights.md` for the result (rejected — OOF
balanced accuracy regressed `-0.0000481`, below the promotion gate) and
`README.md` for how it fits the closed project. Kept as a worked example of
fold-safe target encoding per `docs/0_coding_standards.md`.

This was an execution spec for a coding assistant (Cursor) to implement
directly in `notebooks/02_baseline_modeling.ipynb`. It covers one experiment
only: replacing the hand-picked domain-ordinal categorical maps with proper
fold-safe target encoding, as candidate `lgbm_xgb_target_encoded_ensemble`.

Do not touch v19 (`RUN_GEOMETRY_FORGE`) logic beyond leaving it intact and
disabled/enabled per its own already-recorded decision. v21-v23 (precision
features, logistic diversity, native categorical splits) were separate,
later plans — see `docs/9` for their results.

## 1. Why This Experiment

Every candidate since v7 encodes `diet_type`, `stress_level`, `sleep_quality`,
`physical_activity_level`, and `smoking_alcohol` with a hand-picked domain-order
map in `ORDERED_MAPS` (notebook cell 6). That encodes a *prior belief* about
direction (e.g. `stress_level: high=0, medium=1, low=2`), not the measured
relationship to `health_condition`. `gender` is not encoded into the model at
all today — it is only ever used as a grouping key inside the v19 geometry
forge (`build_geometry_forge`'s `key_groups`), never as a direct numeric
feature for LGBM/XGB. Confirm this yourself before writing code: `gender` does
not appear in `ORDERED_MAPS`, and `DOMAIN_NUMERIC_COLS` /
`GEOMETRY_NUMERIC_COLS` are both built from `select_dtypes(include=[np.number,
'bool'])`, which drops raw categoricals. Target encoding is therefore also the
first time `gender` reaches the model as a feature.

v10, v13, v14, v15, v16, v17, v18 all reweighted or recombined models trained
on this same domain feature set and all failed the `+0.0002` balanced-accuracy
gate (see `docs/9_leaderboard_improvement_insights.md`). This is the first
experiment that changes what the categorical *features themselves* encode,
which is the only lever the ledger hasn't tried yet on the categorical side.

## 2. Constraint: Target Encoding Cannot Reuse the Existing Feature-Matrix Pattern

`build_domain_features` and `build_geometry_forge` (cell 6) are target-free:
they can be computed once, outside any CV loop, because nothing in them looks
at `y`. Target encoding is different — it must be fit on training-fold rows
only, per `docs/0_coding_standards.md`'s existing rule ("Keep feature
engineering fold-safe: target-derived transformations must be fit inside each
training fold only"). This means:

- You **cannot** precompute a static `X_target_encoded` DataFrame before the
  CV loop the way `X_domain` / `X_geometry` are precomputed.
- You **must** write a new evaluation function, parallel to
  `evaluate_lgbm_xgb_ensemble` (cell 12), that computes the target encoding
  *inside* each fold iteration, using that fold's `trn_idx` only, before
  fitting LGBM/XGB. Do not attempt to bolt this onto the existing function by
  passing in a pre-encoded matrix — that would leak.
- Test-set encoding: accumulate a fold-averaged encoding for the test set
  (fit on each fold's full training-fold rows, transform test, average across
  folds — same accumulation style already used for `test_lgb += ... /
  split_count` in `evaluate_lgbm_xgb_ensemble`), so test rows never influence
  their own encoding and the procedure mirrors the model-averaging already in
  the notebook.

## 3. New Helper: Fold-Safe Target Encoder

Add near `ORDERED_MAPS` in cell 6 (or a new cell directly after it):

```python
TARGET_ENCODE_COLS = [
    'diet_type', 'stress_level', 'sleep_quality',
    'physical_activity_level', 'smoking_alcohol', 'gender',
]
TARGET_ENCODE_SMOOTHING = 20.0


def fit_target_encoding(cat_series: pd.Series, y_trn_enc: np.ndarray, n_classes: int, smoothing: float):
    """Return per-class smoothed target-rate maps fit on training-fold rows only."""
    df = pd.DataFrame({'cat': cat_series.astype(str).fillna('__missing__').values})
    global_means = np.bincount(y_trn_enc, minlength=n_classes) / len(y_trn_enc)
    maps = []
    counts = df.groupby('cat').size()
    for k in range(n_classes):
        class_hits = df.assign(hit=(y_trn_enc == k).astype('float64')).groupby('cat')['hit'].sum()
        smoothed = (class_hits + smoothing * global_means[k]) / (counts + smoothing)
        maps.append(smoothed)
    return maps, global_means


def apply_target_encoding(cat_series: pd.Series, maps: list, global_means: np.ndarray) -> np.ndarray:
    values = cat_series.astype(str).fillna('__missing__')
    out = np.zeros((len(values), len(maps)), dtype='float64')
    for k, m in enumerate(maps):
        out[:, k] = values.map(m).fillna(global_means[k]).values
    return out
```

Notes for the implementer:

- `smoothing=20.0` is a starting point (roughly the size of a small category
  level in this ~690k-row train set); treat it as a tunable and sweep at least
  `[5, 20, 50, 100]` once the pipeline works, selecting by OOF balanced
  accuracy the same way `evaluate_lgbm_xgb_ensemble` already sweeps the LGBM
  blend weight.
- Unseen category values at val/test time fall back to `global_means`, computed
  from the training fold only — never from val/test rows.
- This produces 3 encoded columns per categorical (one per class in
  `classes`), named e.g. `stress_level__te__at-risk`,
  `stress_level__te__fit`, `stress_level__te__unhealthy`. Keep all 3 per
  column rather than dropping one for collinearity — tree models handle this
  fine and it keeps the encoder symmetric and simple to reason about.

## 4. New Evaluation Function

Add after `evaluate_lgbm_xgb_ensemble` (cell 12), following its exact shape and
return signature so it slots into the same downstream diagnostics/champion
pattern:

```python
def evaluate_lgbm_xgb_target_encoded_ensemble(
    X_train_base: pd.DataFrame,   # non-categorical numeric features to keep (e.g. DOMAIN_NUMERIC_COLS minus the *_ord columns being replaced, or the full domain set if augmenting)
    X_test_base: pd.DataFrame,
    train_cat_frame: pd.DataFrame,  # raw categorical columns, TARGET_ENCODE_COLS, train rows
    test_cat_frame: pd.DataFrame,   # raw categorical columns, TARGET_ENCODE_COLS, test rows
    label: str = 'lgbm_xgb_target_encoded',
    smoothing: float = TARGET_ENCODE_SMOOTHING,
    lgbm_params=None,
    xgb_params=None,
    random_state=None,
    n_splits=None,
):
    ...
```

Inside, replicate the fold loop from `evaluate_lgbm_xgb_ensemble` but, at the
top of each fold (before building `X_trn`/`X_val`):

1. Call `fit_target_encoding` once per column in `TARGET_ENCODE_COLS` using
   `train_cat_frame.iloc[trn_idx]` and `y_trn_enc`.
2. Build the encoded train/val/test blocks with `apply_target_encoding` and
   concatenate them (`np.hstack` or `pd.concat`) onto `X_train_base` /
   `X_test_base` for that fold.
3. Accumulate the test-side encoded columns fold-averaged, exactly like
   `test_lgb` / `test_xgb` already accumulate `/ split_count`, so the final
   test matrix used for the reported `test_pred` is the fold-averaged
   encoding, not a single fold's encoding.
4. Everything after that (LGBM fit, XGB fit, blend-weight sweep, return tuple)
   should mirror `evaluate_lgbm_xgb_ensemble` exactly — reuse `make_lgbm_model`
   / `make_xgb_model` unchanged.

Keep the same return tuple shape as `evaluate_lgbm_xgb_ensemble` so
`display_diagnostics(...)` and the champion-selection cell can consume it
without modification.

## 5. Feature-Set Variant to Run

Run **one** primary candidate (augment, not replace):

- `X_train_base` = `X_domain[DOMAIN_NUMERIC_COLS]` (the existing v8 domain
  numeric set, unchanged, including the existing `*_ord` columns) plus the new
  target-encoded columns appended.
- Candidate name: `lgbm_xgb_target_encoded_ensemble`.

This keeps the existing domain-ordinal signal intact (in case it captures
something the smoothed target rate doesn't at low sample counts) while adding
the new lever. Do not build a "replace ordinal maps entirely" variant in this
pass — that is an unnecessary second axis of comparison for a first test; if
the augmented candidate clears the gate, a follow-up notebook can test
dropping the ordinal columns as a leaner variant.

## 6. Wiring Into the Notebook

- Add a config flag next to the other `RUN_*` flags in cell 2:
  `RUN_TARGET_ENCODING = True`, and a one-line comment updating the "v19:
  synthetic-geometry feature forge..." comment to mention v20.
- Add a new markdown cell `## 7g. Fold-Safe Target-Encoded LGBM/XGB Ensemble`
  after the existing `## 7f. Synthetic-Geometry LGBM/XGB Ensemble` section
  (after cell 24), following the same prose pattern as 7f's markdown cell
  (one paragraph on the hypothesis, one line naming the candidate).
- Add the code cell in the same `if RUN_TARGET_ENCODING: ... else: <fall back
  to previous champion's oof/test values> ...` shape as cell 24, so the
  notebook still runs end-to-end with the flag off.
- Write `target_encoding_blend_results.csv` to `WORK_DIR`, matching
  `geometry_blend_results.csv`'s pattern in cell 24.

## 7. Champion Selection And Gate

Whatever the champion-selection cell currently compares (check cell 27
onward — it should already be comparing `geometry_oof` / `ensemble_oof` /
`calibrated` variants by balanced accuracy, macro F1, and prediction mix), add
`target_encoded_oof` / `target_encoded_test_pred` as one more candidate in that
same comparison table, not a separate ad hoc check.

Apply the existing, unchanged promotion rule from
`docs/7_submission_quota_strategy.md`:

- OOF balanced accuracy must improve by **at least `+0.0002`** over whichever
  candidate is the locked champion at run time (v8 unless v19 has since been
  promoted — check `docs/6_submission_manifest.md` before running this, since
  it may have changed since this plan was written).
- Macro F1 must not fall.
- `fit` / `unhealthy` prediction shares must not collapse toward train
  prevalence (5.77% / 8.36%) — compare against the current champion's test
  prediction mix, not just eyeball it.
- Only if all three hold does this become a submission candidate; if not,
  record the OOF numbers and reject, same as every prior rejected version.

## 8. After The Run — Doc Updates

Whether it passes or fails the gate:

1. Add a "V20 Fold-Safe Target Encoding Review" section to
   `docs/9_leaderboard_improvement_insights.md`, matching the exact table
   format used for V15–V18 (candidate, balanced accuracy, gain vs champion,
   macro F1 gain, gate column) plus one sentence on the smoothing value
   selected and whether `gender` carried measurable importance.
2. If submitted, log it in `docs/6_submission_manifest.md` (date, file,
   hypothesis, CV score, public score, decision) and update the public
   notebook version reference in the same file.
3. Update `README.md`'s "Modeling Direction" numbered list the same way prior
   versions have been appended/annotated.
4. Do not create a new public Kaggle notebook — this stays inside
   `notebooks/02_baseline_modeling.ipynb` per the existing notebook-first
   convention.

## 9. Explicit Non-Goals For This Pass

- No synthetic-generator artifact mining (duplicate rows, rounding patterns) —
  that is a separate, later plan.
- No non-tree model (MLP/k-NN) — separate, later plan.
- No pseudo-labeling.
- No change to `N_SPLITS`, the multi-seed logic, or any already-disabled
  `RUN_*` flag from v14–v18; leave those exactly as they are.
