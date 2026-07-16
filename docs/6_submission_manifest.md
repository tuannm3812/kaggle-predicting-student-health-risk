# Submission Manifest

Record every leaderboard submission here.

| Date | File | Hypothesis | CV Score | Public Score | Decision |
| --- | --- | --- | ---: | ---: | --- |
| 2026-07-09 | `/private/tmp/student-health-risk-baseline-v3/submission.csv` | First public v3 class-balanced HGB sanity baseline | `0.8785` accuracy / `0.7610` macro F1 | `0.90603` | Keep as baseline; build stronger GBDT next |
| 2026-07-10 | `/private/tmp/student-health-risk-baseline-v5/submission.csv` | Public v5 unweighted HGB selected from compact weighting comparison | `0.96519` accuracy / `0.90275` macro F1 | `0.85038` | Reject; local accuracy misaligned with leaderboard |
| 2026-07-10 | `/private/tmp/student-health-risk-v8-output/submission.csv` | Public v8 balanced LGBM/XGB domain-feature ensemble from notebook output | `0.93746` accuracy / `0.86308` macro F1 / `0.94968` balanced accuracy | `0.94959` | New champion; close to public reference score |

## Public Kaggle Notebooks

| Notebook | Kaggle URL | Status |
| --- | --- | --- |
| Student Health Risk - EDA | https://www.kaggle.com/code/tuannm3812/student-health-risk-eda | Version 3 running with structured markdown |
| Student Health Risk - Baseline Modeling | https://www.kaggle.com/code/tuannm3812/student-health-risk-baseline-modeling | Version 17 complete; 5-fold / CatBoost / cross-fit threshold OOF fails gate; v8 champion `0.94959` remains locked |
