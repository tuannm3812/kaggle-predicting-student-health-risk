# Submission Manifest

Record every leaderboard submission here.

| Date | File | Hypothesis | CV Score | Public Score | Decision |
| --- | --- | --- | ---: | ---: | --- |
| 2026-07-09 | `/private/tmp/student-health-risk-baseline-v3/submission.csv` | First public v3 class-balanced HGB sanity baseline | `0.8785` accuracy / `0.7610` macro F1 | `0.90603` | Keep as baseline; build stronger GBDT next |
| 2026-07-10 | `/private/tmp/student-health-risk-baseline-v5/submission.csv` | Public v5 unweighted HGB selected from compact weighting comparison | `0.96519` accuracy / `0.90275` macro F1 | `0.85038` | Reject; local accuracy misaligned with leaderboard |

## Public Kaggle Notebooks

| Notebook | Kaggle URL | Status |
| --- | --- | --- |
| Student Health Risk - EDA | https://www.kaggle.com/code/tuannm3812/student-health-risk-eda | Version 3 running with structured markdown |
| Student Health Risk - Baseline Modeling | https://www.kaggle.com/code/tuannm3812/student-health-risk-baseline-modeling | Next version prepared with balanced LGBM/XGB domain ensemble |
