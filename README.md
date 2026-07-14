# Predicting Student Health Risk

Kaggle Playground Series S6E7 project for predicting student health risk:
https://www.kaggle.com/competitions/playground-series-s6e7

This repository uses a public-notebook-first Kaggle workflow. The notebooks
generate reproducible outputs, while `docs/` records the modeling rationale,
validation checks, leaderboard submissions, and next-step strategy.

## Current Result

| Model / Notebook | Public Score | Status |
| --- | ---: | --- |
| Balanced LGBM/XGB domain-feature ensemble, public notebook v8 | `0.94959` | Current champion |
| Class-balanced HGB baseline, public notebook v3 | `0.90603` | Former baseline |
| Unweighted HGB, public notebook v5 | `0.85038` | Rejected |

Current champion notebook:
https://www.kaggle.com/code/tuannm3812/student-health-risk-baseline-modeling

## What Worked

- **Domain-ordered categorical encoding** for stress, sleep quality, activity,
  diet, and smoking/alcohol.
- **Row-safe feature engineering** around sleep, activity, BMI, stress, and
  lifestyle signals.
- **Class-balanced tree models** to preserve `fit` and `unhealthy` sensitivity.
- **LightGBM/XGBoost probability blending** selected from OOF validation.
- **Notebook-generated submissions** for reproducibility and public review.

## Repository Structure

- `docs/`: competition notes, EDA findings, modeling decisions, submission
  manifest, and next-step strategy.
- `notebooks/`: executable Kaggle/local notebooks.
- `data/`: local competition files, intentionally ignored.
- `predictions/`: OOF/test prediction matrices, intentionally ignored.
- `scratch/`: temporary helper scripts and automation, intentionally ignored.

## Modeling Direction

The current strategy is to keep the public baseline notebook as the main
submission artifact and make small, auditable improvements:

1. focused LGBM/XGB hyperparameter search with GPU (v15 OOF failed gate; not submitted);
2. targeted interaction features (v14 OOF failed champion gate; not submitted);
3. HGB + LGBM/XGB probability blending (v13 OOF failed; not submitted);
4. probability/class-prior calibration (v10 reviewed, not submitted);
5. CatBoost signal-engine micro-flips (OOF-gated, champion gate applies).

See `docs/6_submission_manifest.md` and
`docs/9_leaderboard_improvement_insights.md` for the current leaderboard record
and improvement plan.
