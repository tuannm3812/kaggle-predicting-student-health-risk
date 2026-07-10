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

The public baseline notebook now keeps HGB as an anchor and adds a stronger
LGBM/XGB ensemble candidate:

- row-safe domain features for sleep, activity, stress, BMI, and lifestyle;
- domain-ordered categorical encodings for health-direction categories;
- a fully balanced `HistGradientBoostingClassifier` anchor;
- balanced LightGBM plus balanced XGBoost probability blending;
- 3-fold stratified validation for public-notebook runtime;
- accuracy, balanced accuracy, macro F1, weighted F1, prediction mix,
  confusion matrix, per-class classification report, and blend-weight results.

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

- LightGBM/XGBoost should improve macro F1 or balanced accuracy, not only plain
  accuracy.
- The prediction share should not collapse almost entirely into `at-risk`.
- Minority-class recall for `fit` and `unhealthy` should be inspected before
  submitting.

## Current Baseline Review

The public v3 baseline notebook completed successfully and generated a valid
`submission.csv`.

Cross-validation summary from the completed Kaggle run:

| Metric | Mean / Overall |
| --- | ---: |
| Accuracy | `0.8785` |
| Balanced accuracy | `0.9091` |
| Macro F1 | `0.7610` |
| Weighted F1 | `0.8910` |

Per-class OOF recall:

| Class | Recall | Precision | F1 |
| --- | ---: | ---: | ---: |
| `at-risk` | `0.8701` | `0.9875` | `0.9251` |
| `fit` | `0.9226` | `0.5055` | `0.6531` |
| `unhealthy` | `0.9345` | `0.5657` | `0.7048` |

The class-balanced HGB baseline intentionally trades majority-class accuracy
for much higher minority-class recall. This is useful if the competition metric
rewards class balance, but it may be too aggressive if the official leaderboard
metric is plain accuracy.

Submission prediction mix:

| Predicted class | Share |
| --- | ---: |
| `at-risk` | `75.71%` |
| `unhealthy` | `13.90%` |
| `fit` | `10.39%` |

Decision: good enough for a first leaderboard calibration submission, not a
champion candidate. The refined baseline notebook now compares unweighted,
lightly weighted, and fully balanced HGB candidates in the same public
notebook. This avoids creating extra public notebooks just to answer the class
weighting question.

## HGB Weighting Comparison Review

The public v5 baseline notebook compared three HGB class-prior strategies in
the same public notebook.

| Candidate | Accuracy | Balanced Accuracy | Macro F1 | Test `at-risk` | Test `fit` | Test `unhealthy` |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| **`hgb_unweighted`** | **`0.96519`** | `0.85734` | `0.90275` | `88.61%` | `4.77%` | `6.62%` |
| `hgb_light_weight` | `0.96438` | `0.87067` | **`0.90294`** | `87.86%` | `5.01%` | `7.13%` |
| `hgb_balanced` | `0.87850` | **`0.90907`** | `0.76104` | `75.71%` | `10.39%` | `13.90%` |

Initial local insight: the fully balanced model over-corrected the class
imbalance, while the unweighted model was the best accuracy-oriented local
baseline.

Leaderboard correction: `hgb_unweighted` scored only `0.85038`, below the
balanced baseline's `0.90603`. Local CV accuracy was therefore misleading for
leaderboard selection. For now, `hgb_balanced` remains the leaderboard champion
and future candidates should preserve class-balanced behavior as a primary
guardrail.

## Public Notebook V7 Direction

The next public notebook improves the baseline in one focused direction:

- keep `hgb_balanced_domain` as the continuity anchor;
- add **domain-ordered features** instead of raw label-only categoricals;
- train **balanced LGBM** and **balanced XGB** on numeric domain features;
- sweep LGBM/XGB probability blend weights by OOF balanced accuracy;
- write `baseline_model_comparison.csv`, `blend_weight_results.csv`,
  `champion_oof_predictions.csv`, and `submission.csv`.

This borrows the useful modeling lesson from a strong public notebook, but uses
our own validation discipline, feature names, diagnostics, and champion
selection rule.
