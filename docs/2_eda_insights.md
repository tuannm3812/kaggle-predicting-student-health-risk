# EDA Insights

Initial data inspection is complete.

## Dataset Shape

| Split | Rows | Columns |
| --- | ---: | ---: |
| Train | 690,088 | 15 |
| Test | 295,753 | 14 |
| Sample submission | 295,753 | 2 |

## Target

The target column is `health_condition`.

| Class | Count | Share |
| --- | ---: | ---: |
| `at-risk` | 592,561 | 85.87% |
| `unhealthy` | 57,724 | 8.36% |
| `fit` | 39,803 | 5.77% |

This is a strongly imbalanced 3-class classification task. Validation should
track both the competition metric and class-balanced diagnostics such as macro
F1, balanced accuracy, and per-class recall. The rare `fit` class is likely to
be easy to under-predict if optimizing plain accuracy only.

Modeling implication: the first baseline should not optimize or report plain
accuracy alone. Accuracy is useful as a sanity check, but it can hide failures
on `fit` and `unhealthy`.

## Features

Numeric:

- `sleep_duration`
- `heart_rate`
- `bmi`
- `calorie_expenditure`
- `step_count`
- `exercise_duration`
- `water_intake`

Categorical:

- `diet_type`
- `stress_level`
- `sleep_quality`
- `physical_activity_level`
- `smoking_alcohol`
- `gender`

Missingness is meaningful: 449,496 missing train cells and 192,642 missing test
cells. Missing-value indicators should be tested as a feature family.

Modeling implication: use imputation plus explicit missing indicators in the
baseline. Later notebooks should test whether missingness itself separates
health conditions, especially for lifestyle fields such as stress, alcohol, and
sleep.

## Public Notebook Structure

`notebooks/01_eda.ipynb` now follows this reader-facing flow:

1. Setup and robust Kaggle/local input discovery.
2. File loading and schema confirmation.
3. Target class balance.
4. Numeric and categorical feature grouping.
5. Missingness profile and missing-count-by-target diagnostics.
6. Train/test drift quick screen.
7. Modeling implications.

This structure is intentionally public-notebook friendly: a Kaggle reader can
skim the markdown and understand why the next baseline uses stratified folds,
class-balanced diagnostics, and missing-value indicators.

## Baseline Direction

`notebooks/02_baseline_modeling.ipynb` now uses a structured baseline:

- median imputation plus missing indicators for numeric features;
- most-frequent imputation plus ordinal encoding for categoricals;
- `HistGradientBoostingClassifier` with class-balanced weighting;
- 5-fold stratified cross-validation;
- accuracy, balanced accuracy, macro F1, weighted F1, classification report,
  and confusion matrix;
- majority-vote test predictions for the first `submission.csv`.

This is a reliable reference point, not the final modeling direction. The next
serious notebook should compare CatBoost, LightGBM, and XGBoost against this
baseline and save OOF probability matrices.

## First EDA Checklist

- Check IDs and duplicates.
- Compare train/test distributions for every feature.
- Inspect missingness by target and by row.
- Identify categorical, ordinal, binary, and continuous fields.
- Inspect rare health-risk labels or high-cost classes against the official
  metric.
- Look for synthetic-data artifacts: clipped values, repeated rows, integer
  coded categories, and train/test drift.
- Document the most plausible feature interactions for student health risk.

## Likely Domain Feature Themes

Validate these from actual columns before using them:

- Sleep, stress, and physical activity interactions.
- Diet, BMI, water intake, and lifestyle clusters.
- Exercise duration, step count, and calorie expenditure consistency.
- Demographics only if they are present and permitted by the competition rules.
- Ratios or flags for clinically meaningful thresholds, kept row-safe.
