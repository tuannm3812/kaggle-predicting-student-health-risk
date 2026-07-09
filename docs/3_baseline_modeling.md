# Baseline Modeling

## Goal

Create a reliable local validation baseline before spending leaderboard
submissions.

## Baseline Ladder

| Step | Model | Purpose |
| --- | --- | --- |
| 1 | Dummy / majority or prior baseline | Sanity check metric and submission format |
| 2 | Logistic Regression or Ridge Classifier | Linear reference and preprocessing check |
| 3 | HistGradientBoosting | Fast sklearn tree baseline |
| 4 | LightGBM | Strong tabular baseline with class weights |
| 5 | XGBoost | Diversity check against LightGBM |
| 6 | CatBoost | Strong handling of categorical features |

## Validation

Use stratified folds for classification unless the target is continuous or
otherwise specified by the competition metric.

For S6E7, use `health_condition` as the target and stratify over:

- `at-risk`
- `unhealthy`
- `fit`

Track:

- Primary competition metric.
- Per-class recall/F1 if classification.
- Log loss or probability calibration if submission accepts probabilities.
- Confusion matrix and rare/high-risk class behavior.
- OOF predictions for every serious model.

## Promotion Rule

Promote a baseline only when it improves cross-validation and preserves sane
class/prediction distribution. Do not choose a model from public leaderboard
score alone.
