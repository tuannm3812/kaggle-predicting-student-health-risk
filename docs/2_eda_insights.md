# EDA Insights

This file will be filled after downloading `train.csv`, `test.csv`, and
`sample_submission.csv`.

## First EDA Checklist

- Confirm target column and class/cardinality.
- Check train/test shape, IDs, duplicates, and missing values.
- Compare train/test distributions for every feature.
- Identify categorical, ordinal, binary, and continuous fields.
- Check target distribution and decide whether stratified CV is required.
- Inspect rare health-risk labels or high-cost classes if the metric is macro
  or balanced.
- Look for synthetic-data artifacts: clipped values, repeated rows, integer
  coded categories, and train/test drift.
- Document the most plausible feature interactions for student health risk.

## Likely Domain Feature Themes

Validate these from actual columns before using them:

- Sleep, stress, and physical activity interactions.
- Diet, BMI, screen time, and lifestyle clusters.
- Attendance, academic pressure, and workload signals.
- Demographics only if they are present and permitted by the competition rules.
- Ratios or flags for clinically meaningful thresholds, kept row-safe.
