# Coding Standards

## Baseline

This project follows the shared `coding-standards/coding_standards.md` at the
GitHub root (`/Users/tuannm3812/Documents/GitHub/coding-standards`) as its
baseline. That file is the fallback for anything not overridden below —
commit message convention, pre-commit/pre-push workflow, feature-engineering
and leakage-prevention rules, and general documentation style all live there.
Everything in this doc is either a project-specific addition or an explicit
override of the shared baseline; keep it that way rather than re-copying
sections that already match the shared file, so the two don't drift silently.

## Repository Scope

Use a notebook-first Kaggle workflow with lightweight documentation:

- `notebooks/` for EDA, baseline modeling, tuning, and final submission logic,
  plus `notebooks/kernels/<name>/` holding each notebook's Kaggle
  `kernel-metadata.json` (see "Pushing Notebooks To Kaggle" below).
- `docs/` for durable findings and decisions.
- `assets/` for README images.
- `scripts/` for small CLI helpers (e.g. the Kaggle push script), not core
  logic.
- `data/` for local Kaggle files. Raw data is ignored.
- `predictions/` for OOF and test predictions. Generated predictions are
  ignored.
- `scratch/` for temporary automation scripts. Generated helpers are ignored.

`data/`, `predictions/`, and `scratch/` are intentional additions on top of
the shared baseline's minimal root (which avoids these by default) — this
project genuinely needs local Kaggle CLI downloads and OOF artifact review,
so keep them, gitignored, rather than removing them to match the generic
default.

## Document Naming

Reserve a new number for a promoted, project-owned finding or decision — not
every parameter tweak or rejected OOF variant (those stay inside
`docs/9_leaderboard_improvement_insights.md`'s running ledger). Current docs:

- `0_coding_standards.md`
- `1_instructions.md`
- `2_eda_insights.md`
- `3_baseline_modeling.md`
- `4_model_optimization_and_ensemble.md`
- `5_implementation_plan.md`
- `6_submission_manifest.md`
- `7_submission_quota_strategy.md`
- `8` — retired. Duplicated `docs/3`'s v3/v5 validation tables and v7/v8
  direction under a different narrative framing; its few unique facts (OOF
  vs. test prediction mix, the actual v7 feature list) were folded into
  `docs/3` and the file was deleted. Number left unused rather than
  renumbering `9`/`10`, which are referenced throughout the project.
- `9_leaderboard_improvement_insights.md`
- `10_v20_target_encoding_plan.md`: a pre-written implementation spec, not a
  result doc — v20 was rejected like every other post-v8 experiment
  (`docs/9`). It exists because target encoding is target-derived and
  couldn't reuse the notebook's existing target-free feature pattern, so it
  needed a genuinely new fold-safe evaluation function spelled out before
  implementation, unlike v19/v21/v22/v23 (target-free or a drop-in model
  swap), which needed only a `docs/9` ledger entry. Reserve a further `1N_*`
  doc only when an experiment needs this kind of pre-implementation spec —
  most rejected OOF variants belong in `docs/9`'s running ledger instead.

Notebook naming: `01_eda.ipynb`, `02_baseline_modeling.ipynb`. Every
experiment since v3 (calibration, blends, HP search, multi-seed, 5-fold,
CatBoost diversity, stacking, geometry forge, target encoding) has lived as a
config-flagged section inside `02_baseline_modeling.ipynb` rather than a new
notebook file — that follows the shared baseline's own rule ("reserve a new
number for a promoted workflow, not every tweak") and should keep being the
default. Only split out `03_model_tuning_and_ensemble.ipynb` /
`04_hyperparameter_tuning.ipynb` if `02` becomes too large or slow to run as
a single public notebook.

## Python Style

- Follow PEP 8: 4-space indentation, group imports stdlib → third-party →
  local with a blank line between groups.
- Prefer small reusable functions with type hints; use Google-style
  docstrings for anything reused across cells.
- Keep feature engineering fold-safe: target-derived transformations
  (target encoding, calibration, thresholding) must be fit inside each
  training fold only — see `docs/10_v20_target_encoding_plan.md` for the
  worked example of what this means in practice for this notebook.

## Notebook Style

Each notebook should include:

- Purpose statement.
- Configuration cell near the top, including explicit mode flags where
  behavior differs by run (this notebook's `RUN_GEOMETRY_FORGE`,
  `RUN_HP_SEARCH`, `RUN_MULTI_SEED`, etc. pattern — keep using one flag per
  experiment rather than commenting code in/out).
- Deterministic seed.
- Markdown insight cells after every important plot or metric.
- Relative input-path handling for Kaggle and local execution.
- Numbered sections with clear reader-facing headers.
- A short interpretation after every major EDA or validation block.
- A final "next moves" section that converts findings into experiments.
- Public-notebook polish: concise prose, no debug clutter, and output tables
  that can be read without opening the source code.

**Outputs policy:** clear outputs before committing if the notebook code
changed and hasn't been rerun on Kaggle yet — don't commit stale results.
**Offline-safety:** the submitted notebook must not depend on internet
access beyond the Kaggle-provided input mount; gate any exploratory package
install behind a config flag, defaulting off.

For public Kaggle notebooks, prefer this section flow:

1. Problem framing and notebook goal.
2. Setup and robust input discovery.
3. Data loading and schema confirmation.
4. Target or metric framing.
5. Feature/missingness/drift diagnostics.
6. Modeling or experiment block.
7. Validation results and interpretation.
8. Next actions.

## Feature Engineering & Leakage Prevention

- Only use fields available at inference time / present in both train and
  test.
- Never derive a feature from the target outside cross-validation folds.
  Target encoding, calibration, and cross-fitted thresholds must be fit
  inside each fold, applied to that fold's held-out rows and to the test
  set, and never fit on the full training set before splitting.
- `build_domain_features` and `build_geometry_forge` in
  `notebooks/02_baseline_modeling.ipynb` are target-free and safe to compute
  once outside the CV loop. Target encoding is not — see
  `docs/10_v20_target_encoding_plan.md` §2 for why it needs its own
  per-fold evaluation function instead of reusing that pattern.

## Plot Style

Use `viridis` as the default color palette or colormap, matching the
previous episode repos.

Plots should answer a specific question. Avoid decorative charts; every
chart should be paired with a concise explanation of what it changes about
modeling or validation.

## Documentation Style

- Findings and implications first, evidence after.
- Cite exact metrics, never vague claims ("balanced accuracy improved by
  `+0.000041`", not "improved slightly").
- Timestamp any fact that can change: leaderboard scores, champion version,
  submission quota, deadlines.
- Keep the broad narrative in `README.md`; put the evidence trail in the
  numbered `docs/` files.

## Git Hygiene

Do not commit raw Kaggle data, model dumps, prediction arrays, local
credentials, temporary scripts, notebook checkpoints, or ad hoc experiment
dumps from `scratch/`.

## Commit Message Convention

Use Conventional Commits, scoped and imperative:

```
<type>(<scope>): <imperative summary>
```

Common types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`. One
coherent change per commit — don't mix a notebook behavior change with an
unrelated docs update unless they're genuinely the same closed change. Put
material detail in the commit body: what changed, what was validated on
Kaggle, what wasn't.

Note: commits before this section was added (through 2026-07-19) predate
this convention and were not scoped this way — don't rewrite that history,
just follow the convention going forward.

## Pre-Commit / Pre-Push Workflow

Before staging anything:

1. Run `git status --short` and review every path — don't blind `git add -A`.
2. If notebook code changed, either rerun it on Kaggle or clear its outputs
   before committing.
3. Stage only the intended change; check `git diff --cached --stat` for
   anything unexpected (data files, credentials, generated artifacts).
4. Write the commit message per the convention above.

Before submitting to the leaderboard specifically:

- Confirm the notebook version pushed to Kaggle matches what's recorded in
  `docs/6_submission_manifest.md`.
- Confirm `/kaggle/working/submission.csv` was regenerated by that run, not
  a stale copy.
- Log the result (notebook version, score, date) immediately after it
  completes — don't let a submission go unrecorded.

## Kaggle Submission Method

Prefer submitting via Kaggle's **notebook submission** ("Submit to
Competition" from within the notebook) over uploading a `submission.csv`
generated elsewhere. Kaggle re-executes the notebook end-to-end, which
verifies the leaderboard result actually matches the committed code. See the
shared `coding-standards/coding_standards.md` (§11) in the GitHub root for
the full rule.

Before submitting, confirm the notebook version matches what's recorded in
this project's results doc, and log the submission (version, score, date)
after it completes.

## Pushing Notebooks To Kaggle

Each notebook's Kaggle kernel has its own `kernel-metadata.json` under
`notebooks/kernels/<name>/`. `notebooks/01_eda.ipynb` and
`notebooks/02_baseline_modeling.ipynb` are the single source of truth; the
`.ipynb` copies inside `notebooks/kernels/*/` are gitignored and regenerated
on every push, not maintained by hand.

Push with `scripts/push_kaggle_kernel.sh <eda|baseline>` rather than running
`kaggle kernels push` directly against a hand-copied file — it copies the
current notebook into the right kernel folder first, so the two never drift.

## Kaggle Access Troubleshooting

Reusable diagnosis for CLI/API friction, not specific to this competition —
useful again for this project or the next one.

**Symptom: `kaggle competitions download` returns `403 Forbidden`, but
`kaggle competitions files` works fine with the same credentials.**

- Cause: valid API credentials are not the same as competition access.
  Kaggle requires the account to explicitly accept the competition's rules
  through the web UI before the API will serve the data files, even though
  read-only metadata calls (like listing files) don't require that
  acceptance.
- Fix: open the competition page in a browser, click "Join Competition" /
  accept the rules, then retry the download. No credential change needed.

**Symptom: `kaggle: command not found`.**

- Cause: the Kaggle CLI installed via `pip install --user` isn't on the
  default `PATH` in this environment.
- Fix: use the full path directly, or add it to `PATH` once for the
  session: `/Users/tuannm3812/Library/Python/3.9/bin/kaggle`. Every Kaggle
  command in this project's docs and scripts already uses the full path for
  this reason.

**Verifying access works**, once both are resolved:

```bash
/Users/tuannm3812/Library/Python/3.9/bin/kaggle competitions files <competition-slug>
/Users/tuannm3812/Library/Python/3.9/bin/kaggle competitions download -c <competition-slug> -p data
```

The notebook-based Kaggle kernel pushes also depend on the same rule
acceptance, so if a kernel push starts failing to read competition data,
check rule acceptance first before assuming a code or credential problem.

**Kaggle competition pages are not fetchable by URL.** Both `WebFetch` and a
raw `curl` only return an empty client-rendered shell (confirmed
2026-07-20) — the Overview/Evaluation/Rules text requires a real browser.
The Kaggle CLI/API has no endpoint for that prose either; `kaggle
competitions list/leaderboard` only return structured metadata (deadline,
team count, scores). If official competition text is needed in a project's
`docs/1_instructions.md`, ask the user to paste it rather than attempting
to fetch it.
