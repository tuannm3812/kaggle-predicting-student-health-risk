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

## First EDA Checklist

- Check IDs and duplicates.
- Compare train/test distributions for every feature.
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
