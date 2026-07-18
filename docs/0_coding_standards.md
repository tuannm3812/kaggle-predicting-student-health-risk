# Coding Standards

## Repository Scope

Use a notebook-first Kaggle workflow with lightweight documentation:

- `notebooks/` for EDA, baseline modeling, tuning, and final submission logic.
- `docs/` for durable findings and decisions.
- `data/` for local Kaggle files. Raw data is ignored.
- `predictions/` for OOF and test predictions. Generated predictions are
  ignored.
- `scratch/` for temporary automation scripts. Generated helpers are ignored.

## Document Naming

- `0_coding_standards.md`
- `1_instructions.md`
- `2_eda_insights.md`
- `3_baseline_modeling.md`
- `4_model_optimization_and_ensemble.md`
- `5_implementation_plan.md`
- `6_submission_manifest.md`
- `7_submission_quota_strategy.md`

Notebook naming:

- `01_eda.ipynb`
- `02_baseline_modeling.ipynb`
- `03_model_tuning_and_ensemble.ipynb`
- `04_hyperparameter_tuning.ipynb`

## Python Style

- Follow PEP 8.
- Use 4 spaces for indentation.
- Prefer small reusable functions with type hints.
- Use Google-style docstrings for reusable utilities.
- Keep feature engineering fold-safe: target-derived transformations must be fit
  inside each training fold only.

## Notebook Style

Each notebook should include:

- Purpose statement.
- Configuration cell near the top.
- Deterministic seed.
- Markdown insight cells after important plots or metrics.
- Relative input-path handling for Kaggle and local execution.
- Numbered sections with clear reader-facing headers.
- A short interpretation after every major EDA or validation block.
- A final "next moves" section that converts findings into experiments.
- Public-notebook polish: concise prose, no debug clutter, and output tables
  that can be read without opening the source code.

For public Kaggle notebooks, prefer this section flow:

1. Problem framing and notebook goal.
2. Setup and robust input discovery.
3. Data loading and schema confirmation.
4. Target or metric framing.
5. Feature/missingness/drift diagnostics.
6. Modeling or experiment block.
7. Validation results and interpretation.
8. Next actions.

## Plot Style

Use `viridis` as the default color palette or colormap, matching the previous
episode repos.

Plots should answer a specific question. Avoid decorative charts; every chart
should be paired with a concise explanation of what it changes about modeling
or validation.

## Git Hygiene

Do not commit raw Kaggle data, model dumps, prediction arrays, local
credentials, temporary scripts, or notebook checkpoints.

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
