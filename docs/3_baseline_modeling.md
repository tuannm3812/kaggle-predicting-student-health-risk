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

The current public baseline notebook implements step 3 with a stronger
preprocessing setup than the initial scaffold:

- numeric median imputation with missing-value indicators;
- categorical most-frequent imputation plus ordinal encoding;
- class-balanced `HistGradientBoostingClassifier`;
- 5-fold stratified validation;
- accuracy, balanced accuracy, macro F1, weighted F1, confusion matrix, and
  per-class classification report.

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

Practical promotion threshold for the next notebook:

- CatBoost/LightGBM/XGBoost should improve macro F1 or balanced accuracy, not
  only plain accuracy.
- The prediction share should not collapse almost entirely into `at-risk`.
- Minority-class recall for `fit` and `unhealthy` should be inspected before
  submitting.
