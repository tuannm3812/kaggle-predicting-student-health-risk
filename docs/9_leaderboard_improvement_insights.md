# Leaderboard Improvement Insights

## Current Position

| Candidate | Public Score | Decision |
| --- | ---: | --- |
| `lgbm_xgb_domain_ensemble` | `0.94959` | Current champion |
| `hgb_balanced` | `0.90603` | Former champion |
| `hgb_unweighted` | `0.85038` | Reject |

The public leaderboard top scores are around `0.951`. The v8 domain-feature
LGBM/XGB ensemble closes most of the original gap and is now within roughly
`0.0015` of that reference level.

## Core Lesson

Local CV accuracy is not a reliable selection signal by itself.

The unweighted HGB model had very strong local accuracy (`0.96519`) but scored
only `0.85038` publicly. The balanced HGB model had lower local accuracy
(`0.87850`) but much better public score (`0.90603`). This means public scoring
or public split behavior rewards preserving minority-class decisions more than
our local accuracy table suggested.

Practical rule: every candidate must be judged by:

- class-balanced behavior;
- minority recall;
- prediction mix;
- leaderboard result, if submitted;
- not local accuracy alone.

## EDA Signals That Matter

Several features have clear class separation:

| Feature / group | Pattern |
| --- | --- |
| `stress_level` | `high` maps strongly toward `unhealthy`; `low` has much higher `fit`; `medium` is almost entirely `at-risk`. |
| `physical_activity_level` | `active` has much higher `fit`; `moderate` and `sedentary` are mostly `at-risk`. |
| `sleep_duration` | `fit` has much higher average sleep (`7.95`), `unhealthy` much lower (`5.37`). |
| `step_count` | `fit` is much higher (`11,651`) than `at-risk` (`8,407`) or `unhealthy` (`8,670`). |
| `exercise_duration` | `fit` is much higher (`50.04`) than `at-risk` (`37.97`) or `unhealthy` (`39.04`). |
| `bmi` | `unhealthy` is higher (`24.12`), `fit` lower (`21.83`). |
| `sleep_quality` | `poor` increases `unhealthy`; `good` increases `fit`. |
| `smoking_alcohol` | `yes` increases `unhealthy`; `no` increases `fit`. |

Missingness is frequent but appears intentionally balanced between train and
test. Row-level missing counts are similar by target, so missing indicators are
useful but probably not the main leaderboard lever.

## Why The Current Balanced Model Works Better Publicly

The balanced model predicts many more `fit` and `unhealthy` rows than their
training prevalence:

| Split / model | `at-risk` | `fit` | `unhealthy` |
| --- | ---: | ---: | ---: |
| Train target | `85.87%` | `5.77%` | `8.36%` |
| `hgb_balanced` test | `75.71%` | `10.39%` | `13.90%` |
| `hgb_unweighted` test | `88.61%` | `4.77%` | `6.62%` |

The public score tells us that the unweighted model became too conservative on
minority classes. The balanced model is probably closer to the hidden public
distribution or to the competition metric's implicit class-balance pressure.

## Highest-Value Next Experiments

### 1. Balanced LGBM/XGB Domain Ensemble

Use LightGBM and XGBoost because strong public evidence suggests balanced GBDT
ensembles handle this synthetic tabular problem better than the single HGB
anchor.

Recommended setup:

- **domain-ordered encodings** for stress, sleep quality, activity, diet, and
  smoking/alcohol;
- row-safe lifestyle composites for sleep recovery, activity volume, BMI risk,
  and stress/sleep pressure;
- balanced LightGBM plus balanced XGBoost;
- OOF probability blend sweep instead of a hard-coded blend weight;
- one notebook-generated `submission.csv`.

Promotion gate:

- public score must beat the current `0.94959` champion;
- `fit` and `unhealthy` prediction shares should not collapse;
- local balanced accuracy or macro F1 should stay competitive;
- blend diagnostics should explain the selected LGBM/XGB weight.

### 2. Targeted Interaction Features

Add simple row-safe features before model tuning:

- `sleep_activity_score`: sleep duration plus activity/step/exercise signals;
- `stress_sleep_risk`: high stress plus low sleep or poor sleep quality;
- `activity_intensity`: step count per exercise minute;
- `calorie_per_step`;
- `bmi_sleep_interaction`;
- missing-count feature and selected missing flags.

These are plausible because the EDA shows the target is driven by lifestyle
combinations, not just isolated variables.

### 3. Threshold / Prior Calibration Around Balanced Predictions

Instead of switching to unweighted models, adjust balanced model predictions
more gently:

- start from balanced model probabilities;
- test class multipliers for `fit` and `unhealthy`;
- tune on OOF balanced behavior and prediction mix;
- avoid collapsing minority predictions to train prevalence.

This is safer than the unweighted direction because it keeps the model's useful
minority sensitivity.

## What Not To Do

- Do not submit more raw accuracy-oriented unweighted variants.
- Do not trust local accuracy as the main selector.
- Do not create many public notebooks for small changes.
- Do not add complex feature piles without prediction-mix diagnostics.

## Next Concrete Move

Keep the existing public baseline notebook as the main notebook and make only
small, auditable refinements around the **balanced LGBM/XGB domain ensemble**.
Do not create a new public notebook unless the baseline notebook becomes too
large or slow.

v13–v18 ruled out HGB blending, interaction FE, focused HP search, multi-seed
averaging, 5-fold CV, CatBoost diversity blending, cross-fitted thresholds, and
OOF probability stacking. Stop spending submissions on rounding-level OOF
changes; the next useful move should change the feature surface under the same
champion gate.

## Implementation Status

The public baseline notebook v8 completed and was submitted:

- row-safe lifestyle interaction features;
- balanced HGB with engineered features;
- balanced LGBM/XGB probability ensemble;
- OOF blend-weight sweep by balanced accuracy;
- champion selection by balanced accuracy and macro F1;
- one notebook-generated `submission.csv`.

Result: public score `0.94959`, replacing `hgb_balanced` (`0.90603`) as the
current champion.

## V10 Calibration Sweep Review

Notebook v10 tested a small class-prior calibration sweep on top of the v8
LGBM/XGB probability blend.

| Candidate | Accuracy | Balanced Accuracy | Macro F1 | Test `at-risk` | Test `fit` | Test `unhealthy` |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| `calibrated_lgbm_xgb_domain_ensemble` | `0.93726` | **`0.94970`** | `0.86257` | `80.92%` | `7.40%` | `11.67%` |
| `lgbm_xgb_domain_ensemble` | `0.93746` | `0.94968` | `0.86308` | `80.94%` | `7.37%` | `11.69%` |
| `hgb_balanced_domain` | **`0.93874`** | `0.94932` | **`0.86510`** | `81.06%` | `7.36%` | `11.58%` |

Best calibration setting:

- `fit_multiplier = 1.12`
- `unhealthy_multiplier = 1.00`

Decision: **do not submit**. The balanced-accuracy gain over v8 is only
`+0.000014`, while accuracy and macro F1 both move down. This is too small to
spend a leaderboard submission under the current quota strategy.

## V13 HGB + LGBM/XGB Blend Review

Notebook v13 tested a convex probability blend between the balanced HGB anchor
and the v8 LGBM/XGB ensemble. Best OOF weight was **40% HGB / 60% LGBM-XGB**.

| Candidate | Accuracy | Balanced Accuracy | Macro F1 | Gate |
| --- | ---: | ---: | ---: | --- |
| `calibrated_lgbm_xgb_domain_ensemble` | `0.93726` | **`0.94970`** | `0.86257` | Fail (macro F1 down) |
| `hgb_lgbm_xgb_domain_blend` | `0.93816` | `0.94968` | `0.86428` | Fail (`+0.000003` bal-acc) |
| `lgbm_xgb_domain_ensemble` | `0.93746` | `0.94968` | `0.86308` | Base / keep |
| `catboost_signal_engine` | `0.93746` | `0.94967` | `0.86307` | Fail |

Decision: **do not submit**. The HGB blend gained only `+0.000003` balanced
accuracy versus v8 — far below the `0.0002` champion gate. Same lesson as v10:
rounding-level OOF moves are not worth a leaderboard attempt.

## V14 Targeted Interaction Features Review

Notebook v14 retrained the balanced LGBM/XGB blend on domain features plus a
small interaction pack (`sleep_activity_score`, `stress_sleep_risk`, mismatch /
deficit scores, and key missing flags) as `lgbm_xgb_interaction_ensemble`.

| Candidate | Balanced Accuracy | Gain vs v8 | Macro F1 gain | Gate |
| --- | ---: | ---: | ---: | --- |
| `calibrated_lgbm_xgb_domain_ensemble` | **`0.94970`** | `+0.000015` | `-0.000507` | Fail |
| `lgbm_xgb_interaction_ensemble` | `0.94968` | `+0.000003` | `+0.001221` | Fail |
| `lgbm_xgb_domain_ensemble` | `0.94968` | — | — | Base / keep |
| `catboost_signal_engine` | `0.94968` | `-0.000004` | `+0.000006` | Fail |

Decision: **do not submit**. Interaction extras improved macro F1 slightly but
barely moved balanced accuracy (`+0.000003`), same gate failure mode as v10/v13.

## What To Try Next

v10–v18 all failed the `0.0002` bal-acc gate. The active notebook experiment is
**v19 synthetic-geometry feature forge** (ranks, q-bins, group deviations, rank
composites) retrained with the v8 balanced LGBM/XGB recipe.

Keep `0.94959` public champion locked until a candidate clearly beats the gate.

## V15 Focused HP Search

Notebook v15 enabled GPU (`enable_gpu: true`, XGBoost `device=cuda`), skipped
previously rejected ablations, and searched deeper/longer LGBM/XGB configs on
the domain feature set.

Best tuned blend: **LGBM deeper + XGB v8** at **50/50** weight.

| Candidate | Balanced Accuracy | Gain vs v8 | Macro F1 gain | Gate |
| --- | ---: | ---: | ---: | --- |
| `lgbm_xgb_tuned_domain_ensemble` | **`0.94979`** | `+0.000041` | `+0.002594` | Fail |
| `calibrated_lgbm_xgb_domain_ensemble` | `0.94977` | `+0.000022` | `-0.000299` | Fail |
| `lgbm_xgb_domain_ensemble` | `0.94975` | — | — | Base / keep |

Decision: **do not submit**. The HP search is the strongest OOF lift so far
among failed candidates, but `+0.000041` is still well below the `0.0002` gate.

Next options with a realistic shot at clearing the gate are larger distributional
changes (e.g., different CV scheme / seed averaging with a material bal-acc
lift), not another small local grid around the same recipe.

## V16 Multi-Seed Averaging

Notebook v16 kept the v8 domain LGBM/XGB recipe and averaged component
probabilities across seeds `[42, 0, 7, 17, 99]`, then re-swept the blend weight
on the averaged OOF. Seed `42` reused the already-trained v8 component
probabilities. GPU was available.

Best averaged blend: **70% LGBM / 30% XGB**.

| Candidate | Balanced Accuracy | Gain vs v8 | Macro F1 gain | Gate |
| --- | ---: | ---: | ---: | --- |
| `lgbm_xgb_multiseed_domain_ensemble` | **`0.94977`** | `+0.000029` | `+0.000441` | Fail |
| `calibrated_lgbm_xgb_domain_ensemble` | `0.94977` | `+0.000022` | `-0.000299` | Fail |
| `lgbm_xgb_domain_ensemble` | `0.94975` | — | — | Base / keep |

Decision: **do not submit**. Seed bagging helped slightly but stayed far below
the `0.0002` gate. The notebook correctly kept the v8 champion.

## V17 Five-Fold, CatBoost Diversity, Cross-Fitted Thresholds

Notebook v17 disabled multi-seed averaging and tested three larger changes on
GPU. Champion stayed `lgbm_xgb_domain_ensemble`.

| Candidate | Balanced Accuracy | Gain vs v8 | Macro F1 gain | Gate |
| --- | ---: | ---: | ---: | --- |
| `lgbm_xgb_fivefold_domain_ensemble` | **`0.94977`** | `+0.000028` | `+0.000062` | Fail |
| `crossfit_threshold_lgbm_xgb` | `0.94977` | `+0.000022` | `-0.000299` | Fail |
| `lgbm_xgb_catboost_domain_blend` | `0.94975` | `+0.000002` | `+0.000023` | Fail |
| `lgbm_xgb_domain_ensemble` | `0.94975` | — | — | Base / keep |
| `catboost_native_balanced` | `0.94891` | `-0.000836` | `-0.001369` | Fail |

Details:

- best 5-fold blend remained **50/50 LGBM/XGB**;
- best CatBoost diversity weight was only **0.15**, essentially no lift;
- cross-fitted multipliers consistently chose `fit=1.08`, `unhealthy=1.00`, but
  holdout-safe scoring still failed the gate.

Decision: **do not submit**. These larger validation/distributional changes are
still rounding-level versus v8. The public champion `0.94959` remains locked.

## V18 OOF Probability Stacking

Notebook v18 disabled the failed five-fold and cross-fit threshold sweeps, kept
CatBoost OOF probabilities, and trained a multinomial logistic meta-learner on
stacked base probabilities from LGBM, XGBoost, HGB, and CatBoost. GPU was
available.

Best config: `C=0.5`, `class_weight=balanced`, components
`['catboost', 'hgb', 'lgbm', 'xgb']`.

| Candidate | Balanced Accuracy | Gain vs v8 | Macro F1 gain | Gate |
| --- | ---: | ---: | ---: | --- |
| `calibrated_lgbm_xgb_domain_ensemble` | `0.94977` | `+0.000022` | `-0.000299` | Fail |
| `lgbm_xgb_domain_ensemble` | `0.94975` | — | — | Base / keep |
| `oof_probability_stack` | `0.94962` | `-0.000128` | `+0.000513` | Fail |

Decision: **do not submit**. Stacking hurt balanced accuracy versus v8 while only
improving macro F1. Convex blends and second-level logistic stacking around the
same base models appear saturated.

Next ideas should change the feature surface more aggressively under the same
champion gate.

## V19 Synthetic-Geometry Feature Forge

Notebook v19 adds target-free geometry features inspired by strong public signal-engine
notebooks, without copying their leaderboard flip engine:

- joint `%` ranks and `q32` bins on numeric columns;
- group-median deviations (`num__dev__group`);
- category frequency encodings;
- rank composites (`wellness_rank_core`, `activity_rank_core`, `body_hydration_rank`).

Candidate: `lgbm_xgb_geometry_ensemble` (domain + geometry, same v8 LGBM/XGB recipe).

Promotion rule unchanged: OOF balanced-accuracy gain `>= 0.0002` versus v8, macro F1
must not fall, then public score `> 0.94959`.

## V19 Synthetic-Geometry Feature Forge Review

Notebook v19 ran to completion on Kaggle (GPU, 3-fold, 65 new geometry columns
on top of the 40 domain numeric columns). Best geometry blend was **50% LGBM /
50% XGB**, same weight as v8.

| Candidate | Balanced Accuracy | Gain vs v8 | Macro F1 gain | Gate |
| --- | ---: | ---: | ---: | --- |
| `lgbm_xgb_domain_ensemble` | `0.94975` | — | — | Base / keep |
| `lgbm_xgb_geometry_ensemble` | `0.94972` | `-0.0000227` | `-0.000169` | Fail |
| `hgb_balanced_domain` | `0.94928` | `-0.000467` | `+0.001390` | Fail |

Decision: **do not submit**. Rank geometry, quantile bins, group-median
deviations, and frequency encodings made balanced accuracy and macro F1
*slightly worse*, not better — the first geometry attempt regressed rather
than saturated. `lgbm_xgb_domain_ensemble` (v8) remains champion at `0.94959`
public. `RUN_GEOMETRY_FORGE` is now disabled by default in the notebook.

This is the second feature-surface-level change tried (after v14's targeted
interactions) and the second to fail outright rather than just miss the gate
by a small margin, which weakens the case that "any feature-surface change"
is the fix — the next attempt (v20) should bring a change that plausibly adds
*new information* (measured target-conditional rates, including for `gender`,
which no prior version has fed to the model at all) rather than another
transformation of the same numeric columns.

## V20 Fold-Safe Target Encoding

Notebook v20 replaces the guesswork in `ORDERED_MAPS` with fold-safe target
encoding: for each of `diet_type`, `stress_level`, `sleep_quality`,
`physical_activity_level`, `smoking_alcohol`, and `gender`, encode the
smoothed per-class target rate, fit inside each training fold only (see
`docs/10_v20_target_encoding_plan.md` for the full spec). `gender` reaches
the model as a real feature for the first time. The domain-ordinal columns
are kept alongside the new encoding (augment, not replace) for the first
pass.

Candidate: `lgbm_xgb_target_encoded_ensemble`. Promotion rule unchanged: OOF
balanced-accuracy gain `>= 0.0002` versus v8, macro F1 must not fall, then
public score `> 0.94959`.

## V20 Fold-Safe Target Encoding Review

Notebook v20 ran to completion on Kaggle (GPU, 3-fold). Both smoothing values
in `TARGET_ENCODE_SMOOTHING_GRID` (`20.0`, `50.0`) produced identical results
to 5 decimal places — expected, not a bug: all six target-encoded columns
(`diet_type`, `stress_level`, `sleep_quality`, `physical_activity_level`,
`smoking_alcohol`, `gender`) have only 2-3 levels with hundreds of thousands
of rows each, so a smoothing prior of 20-50 pseudo-observations is negligible
against counts that large. The best blend was **100% LGBM / 0% XGB** — the
first time XGB has contributed zero weight to a champion-track blend.

| Candidate | Balanced Accuracy | Gain vs v8 | Macro F1 gain | Gate |
| --- | ---: | ---: | ---: | --- |
| `lgbm_xgb_domain_ensemble` | `0.94975` | — | — | Base / keep |
| `lgbm_xgb_target_encoded_ensemble` | `0.94970` | `-0.0000481` | `+0.000711` | Fail |
| `hgb_balanced_domain` | `0.94928` | `-0.000467` | `+0.001390` | Fail |

Decision: **do not submit**. Balanced accuracy regressed slightly, same
direction and similar magnitude as v19's geometry forge (`-0.0000227`), but
unlike v19 (macro F1 `-0.000169`), v20's macro F1 clearly *improved*
(`+0.000711`) — the first feature-surface attempt where the two metrics move
in opposite directions instead of together. The champion gate is written
against balanced accuracy only, so v20 still fails it even though a
macro-F1-first reviewer might read this run as a mild improvement.

This is worth flagging rather than just logging as another rejection: if a
future candidate shows this same macro-F1-up / balanced-accuracy-down
trade-off more strongly, it may be worth deciding explicitly whether the
promotion gate should stay balanced-accuracy-only, since the public
leaderboard metric is not confirmed to be balanced accuracy itself (see
"Core Lesson" above — local CV is not a reliable stand-in for the actual
public metric).

`lgbm_xgb_domain_ensemble` (v8) remains champion at `0.94959` public.
`RUN_TARGET_ENCODING` was initially left enabled as reusable scaffolding, but
is now disabled by default (`RUN_TARGET_ENCODING = False`) starting v21 to
keep notebook runtime down, matching how prior rejected sections (HP search,
geometry forge) were disabled once their result was recorded. The encoder
helpers (`fit_target_encoding` / `apply_target_encoding`) remain in the
notebook for reuse.

## V21 Rounding/Precision Artifact Features

Before building another feature-surface experiment, we tested the classic
Playground-Series lever directly against this dataset: **exact and
near-duplicate row detection**. It does not apply here.

- Exact duplicates: **zero**. Grouping the combined train+test table (985,841
  rows) by all 13 feature columns (categoricals + all 7 numeric columns,
  NaNs filled with a sentinel) yields 985,841 distinct groups — every row is
  unique.
- Near duplicates: also effectively absent. Coarsening the numeric columns
  (round `sleep_duration`/`bmi`/`heart_rate`/`exercise_duration` to whole
  numbers, `water_intake` to 1 decimal, bucket `calorie_expenditure` into
  50-unit and `step_count` into 500-unit bins) and regrouping still produces
  985,837 singleton groups out of 985,839 — essentially 100%. Seven
  independent-ish continuous numeric columns are high-dimensional enough that
  rows never collide even under aggressive rounding.

This rules out row-level duplicate mining for this competition; it is not
the lever it is in some other Playground Series entries.

What the same investigation *did* surface: five numeric columns
(`sleep_duration`, `bmi`, `water_intake`, `heart_rate`,
`exercise_duration`) are normally recorded to 1-2 decimal places, but a small
share of rows land on exact whole numbers. In training data:

| Column | Whole-number share | `fit` rate (whole) | `fit` rate (non-whole) |
| --- | ---: | ---: | ---: |
| `sleep_duration` | `0.66%` | `9.26%` | `5.74%` |
| `bmi` | `0.79%` | `8.02%` | `5.75%` |
| `water_intake` | `0.97%` | `5.73%` | `5.77%` |
| `heart_rate` | `10.17%` | `5.94%` | `5.75%` |
| `exercise_duration` | `12.78%` | `4.82%` | `5.91%` |

`sleep_duration` and `bmi` show a `fit` rate roughly double for whole-number
rows; `water_intake`/`heart_rate` show little difference; `exercise_duration`
moves the opposite direction. Mixed, but real enough on two columns to test.
We add `{col}_decimal_places` and `{col}_is_whole` for all five columns
(target-free, row-safe, reuses `evaluate_lgbm_xgb_ensemble` directly since no
fold-safety concern applies) as `lgbm_xgb_precision_ensemble`.

Promotion rule unchanged: OOF balanced-accuracy gain `>= 0.0002` versus v8,
macro F1 must not fall, then public score `> 0.94959`.

## V21 Rounding/Precision Artifact Features Review

Notebook v21 ran to completion on Kaggle (GPU, 3-fold, 10 new precision
columns). Best blend shifted to **70% LGBM / 30% XGB** (every prior version
picked 50/50 or 100/0 LGBM).

| Candidate | Balanced Accuracy | Gain vs v8 | Macro F1 gain | Gate |
| --- | ---: | ---: | ---: | --- |
| `lgbm_xgb_domain_ensemble` | `0.94975` | — | — | Base / keep |
| `lgbm_xgb_precision_ensemble` | `0.94972` | `-0.0000243` | `+0.0000969` | Fail |
| `hgb_balanced_domain` | `0.94928` | `-0.000467` | `+0.001390` | Fail |

Decision: **do not submit**. This is the smallest-magnitude result of the
three feature-surface experiments — essentially flat, not a clear regression
like v19 or v20. `lgbm_xgb_domain_ensemble` (v8) remains champion at
`0.94959` public.

### Pattern across v19-v21

Three structurally different "add genuinely new information" feature
experiments have now all landed in the same narrow band, all failing the
`+0.0002` gate:

| Version | Idea | Bal-acc gain | Macro F1 gain |
| --- | --- | ---: | ---: |
| v19 | Rank/quantile/deviation geometry | `-0.0000227` | `-0.000169` |
| v20 | Fold-safe target encoding | `-0.0000481` | `+0.000711` |
| v21 | Decimal-precision/whole-number flags | `-0.0000243` | `+0.0000969` |

None of these are copies of each other — geometry ranks, measured
target-conditional rates, and generator-precision artifacts are genuinely
different information sources — yet all three move balanced accuracy by
less than `0.00005` in either direction. This is stronger evidence than any
single result that the **balanced LGBM/XGB-on-domain-features recipe itself
has converged to a ceiling around `0.9497-0.9498` balanced accuracy**, not
that the right feature hasn't been found yet. Closing the remaining
`~0.0015` gap to the public top (`~0.951`) likely needs a change that isn't
"one more engineered feature on the same two tree models" — see the updated
recommendations in `README.md`.

