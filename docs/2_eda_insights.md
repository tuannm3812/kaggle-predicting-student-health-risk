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
6. Feature relevance: mutual information.
7. Exact train/test duplicate check.
8. Train/test drift: quick screen + adversarial validation.
9. Modeling implications.

This structure is intentionally public-notebook friendly: a Kaggle reader can
skim the markdown and understand why the next baseline uses stratified folds,
class-balanced diagnostics, and missing-value indicators.

## Feature Relevance (Mutual Information)

Mutual information with `health_condition`, computed on median-imputed
numerics and category-coded categoricals:

| Feature | Mutual information |
| --- | ---: |
| `sleep_duration` | `0.1551` |
| `stress_level` | `0.1426` |
| `physical_activity_level` | `0.0560` |
| `bmi` | `0.0323` |
| `exercise_duration` | `0.0228` |
| `step_count` | `0.0221` |
| `sleep_quality` | `0.0142` |
| `water_intake` | `0.0092` |
| `calorie_expenditure` | `0.0063` |
| `smoking_alcohol` | `0.0057` |
| `heart_rate` | `0.0048` |
| `diet_type` | `0.0002` |
| `gender` | `0.0002` |

`sleep_duration` and `stress_level` lead by a clear margin; `diet_type` and
`gender` carry almost no independent signal. This matches an independent
computation from a top-voted public notebook for this competition
(`georgymamarin/s6e7-quit-chasing-0-950-like-everyone`), which found the same
two features leading and the same two trailing.

## Exact Train/Test Duplicate Check

Some Playground Series entries are generated from a real source table in a
way that leaves test rows byte-identical to a train row across every
feature — in that case the label isn't predicted, it's known outright. This
does **not** apply here: grouping the combined 985,841 train+test rows by
every feature column yields 985,841 distinct groups — zero exact duplicates,
confirmed via two independent implementations (this notebook's groupby check,
and a byte-identical string-join check reproduced from
`lucifer19/student-health-signal-engine`, another top public notebook).

## Train/Test Drift: Adversarial Validation

A classifier trained to distinguish train rows from test rows scores
ROC-AUC `~0.653` (0.50 = indistinguishable) — matching the `~0.65` reported
independently by `georgymamarin/s6e7-quit-chasing-0-950-like-everyone`. This
is a mild multivariate shift invisible to single-feature histograms, driven
mostly by `water_intake`, `physical_activity_level`, `calorie_expenditure`,
and `bmi` (permutation importance). It is nowhere near the `~0.8+` AUC that
would justify distrusting a plain stratified split or importance-weighting
training rows — safe to proceed as this project already has.

## Baseline Direction

**Status: superseded.** This describes the first baseline notebook, before
domain-ordered encoding or the champion LGBM/XGB ensemble existed:

- median imputation plus missing indicators for numeric features;
- most-frequent imputation plus ordinal encoding for categoricals;
- `HistGradientBoostingClassifier` with class-balanced weighting;
- 5-fold stratified cross-validation;
- accuracy, balanced accuracy, macro F1, weighted F1, classification report,
  and confusion matrix;
- majority-vote test predictions for the first `submission.csv`.

The champion recipe that actually shipped (v8 onward, see
`docs/3_baseline_modeling.md`) diverged from this in two ways: domain-ordered
categorical encoding instead of plain ordinal, and 3-fold CV instead of
5-fold (5-fold was retried once at v17 and gained only `+0.000028` balanced
accuracy — see `docs/7`, still not worth the added runtime). CatBoost,
LightGBM, and XGBoost were all compared against this HGB baseline as
originally planned here; the full result is the v3→v8 progression documented
in `docs/3`.

## First EDA Checklist

Pre-project checklist that shaped the actual notebook flow above (each item
maps to a numbered section in `notebooks/01_eda.ipynb`):

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

Pre-project hypotheses, validated against actual columns before use. All five
were confirmed present and became the domain feature set in
`docs/3_baseline_modeling.md`'s champion recipe:

- Sleep, stress, and physical activity interactions.
- Diet, BMI, water intake, and lifestyle clusters.
- Exercise duration, step count, and calorie expenditure consistency.
- Demographics only if they are present and permitted by the competition rules.
- Ratios or flags for clinically meaningful thresholds, kept row-safe.
