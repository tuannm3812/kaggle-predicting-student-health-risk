# Implementation Plan: Predicting Student Health Risk

## 1. Context

This project targets Kaggle Playground Series S6E7:
https://www.kaggle.com/competitions/playground-series-s6e7

The local GitHub repo is currently empty apart from this scaffold. Prior
episode repos show a successful pattern:

- S6E4: compact CatBoost/HGB ensemble plus conservative rare-class threshold.
- S6E5: tuned LightGBM, feature-family validation, calibration diagnostics.
- S6E6: strongest process, with OOF matrices, stacking, public-score-aware
  submission gates, and row-level disagreement analysis.

S6E7 should reuse that discipline with less early complexity: build a clean
single-model baseline first, then only ensemble once OOF evidence supports it.

## 2. Immediate Blocker

Kaggle API credentials are configured and the competition files have been
downloaded locally.

To refresh the files:

```bash
/Users/tuannm3812/Library/Python/3.9/bin/kaggle competitions files playground-series-s6e7
/Users/tuannm3812/Library/Python/3.9/bin/kaggle competitions download -c playground-series-s6e7 -p data
unzip data/playground-series-s6e7.zip -d data
```

Confirmed target: `health_condition`.

Confirmed class distribution:

- `at-risk`: 592,561
- `unhealthy`: 57,724
- `fit`: 39,803

Then update `docs/1_instructions.md`.

## 3. Repository Structure

```text
.
├── README.md
├── requirements.txt
├── .gitignore
├── data/
├── docs/
├── notebooks/
├── predictions/
└── scratch/
```

## 4. Notebook Sequence

| Notebook | Purpose | Output |
| --- | --- | --- |
| `01_eda.ipynb` | Data shape, target, drift, missing values, feature types | `docs/2_eda_insights.md` |
| `02_baseline_modeling.ipynb` | Stratified CV and baseline model comparison | `docs/3_baseline_modeling.md`, OOF predictions |
| `03_model_tuning_and_ensemble.ipynb` | Tune champion models, validate features, blend/threshold | `docs/4_model_optimization_and_ensemble.md`, `submission.csv` |
| `04_hyperparameter_tuning.ipynb` | Optuna or focused searches for top models | `scratch/best_hyperparameters.json` |

## 5. Validation Strategy

Default to stratified 5-fold CV for classification. If the metric is macro F1,
balanced accuracy, or another class-balanced metric, optimize the target class
recalls rather than raw accuracy.

Use:

- `StratifiedKFold(n_splits=5, shuffle=True, random_state=42)`.
- One OOF prediction file per serious model.
- Per-class diagnostics and prediction distribution checks.
- A holdout sanity check only if train size is very large.

## 6. Modeling Strategy

Start with:

1. Simple preprocessing baseline.
2. HistGradientBoosting for speed.
3. LightGBM with categorical handling or encoded categoricals.
4. CatBoost if categorical columns are present.
5. XGBoost for diversity.

Then test:

- Class weights if the metric is class-balanced or the target is skewed.
- Conservative thresholding for high-risk or rare classes.
- Small OOF blend grids.
- Logistic regression meta-learner over model probabilities.

## 7. Submission Policy

Spend submissions like S6E6:

1. Keep a reproducible champion `submission.csv`.
2. Submit only candidates with a written hypothesis.
3. Record every public score in `docs/6_submission_manifest.md`.
4. Stop a branch after a materially worse public result.
5. Avoid repeated alpha/threshold sweeps without new validation evidence.

## 8. First Day Execution Plan

1. Run public Kaggle EDA notebook.
2. Run public Kaggle baseline notebook.
3. Record notebook links and public run status.
4. Build `02_baseline_modeling.ipynb` with at least sklearn HGB, LightGBM, and
   CatBoost.
5. Generate first OOF metrics and a sanity-check submission.
