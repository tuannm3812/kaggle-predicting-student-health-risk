# Leaderboard Improvement Insights

## Current Position

| Candidate | Public Score | Decision |
| --- | ---: | --- |
| `lgbm_xgb_domain_ensemble` | `0.94959` | Current champion |
| `hgb_balanced` | `0.90603` | Former champion |
| `hgb_unweighted` | `0.85038` | Reject |

The public leaderboard top scores are around `0.951`. The v8 domain-feature
LGBM/XGB ensemble closes most of the original gap and is now within roughly
`0.0015` of that reference level.

## Core Lesson

Local CV accuracy is not a reliable selection signal by itself.

The unweighted HGB model had very strong local accuracy (`0.96519`) but scored
only `0.85038` publicly. The balanced HGB model had lower local accuracy
(`0.87850`) but much better public score (`0.90603`). This means public scoring
or public split behavior rewards preserving minority-class decisions more than
our local accuracy table suggested.

Practical rule: every candidate must be judged by:

- class-balanced behavior;
- minority recall;
- prediction mix;
- leaderboard result, if submitted;
- not local accuracy alone.

## EDA Signals That Matter

Several features have clear class separation:

| Feature / group | Pattern |
| --- | --- |
| `stress_level` | `high` maps strongly toward `unhealthy`; `low` has much higher `fit`; `medium` is almost entirely `at-risk`. |
| `physical_activity_level` | `active` has much higher `fit`; `moderate` and `sedentary` are mostly `at-risk`. |
| `sleep_duration` | `fit` has much higher average sleep (`7.95`), `unhealthy` much lower (`5.37`). |
| `step_count` | `fit` is much higher (`11,651`) than `at-risk` (`8,407`) or `unhealthy` (`8,670`). |
| `exercise_duration` | `fit` is much higher (`50.04`) than `at-risk` (`37.97`) or `unhealthy` (`39.04`). |
| `bmi` | `unhealthy` is higher (`24.12`), `fit` lower (`21.83`). |
| `sleep_quality` | `poor` increases `unhealthy`; `good` increases `fit`. |
| `smoking_alcohol` | `yes` increases `unhealthy`; `no` increases `fit`. |

Missingness is frequent but appears intentionally balanced between train and
test. Row-level missing counts are similar by target, so missing indicators are
useful but probably not the main leaderboard lever.

## Why The Current Balanced Model Works Better Publicly

The balanced model predicts many more `fit` and `unhealthy` rows than their
training prevalence:

| Split / model | `at-risk` | `fit` | `unhealthy` |
| --- | ---: | ---: | ---: |
| Train target | `85.87%` | `5.77%` | `8.36%` |
| `hgb_balanced` test | `75.71%` | `10.39%` | `13.90%` |
| `hgb_unweighted` test | `88.61%` | `4.77%` | `6.62%` |

The public score tells us that the unweighted model became too conservative on
minority classes. The balanced model is probably closer to the hidden public
distribution or to the competition metric's implicit class-balance pressure.

## Highest-Value Next Experiments

### 1. Balanced LGBM/XGB Domain Ensemble

Use LightGBM and XGBoost because strong public evidence suggests balanced GBDT
ensembles handle this synthetic tabular problem better than the single HGB
anchor.

Recommended setup:

- **domain-ordered encodings** for stress, sleep quality, activity, diet, and
  smoking/alcohol;
- row-safe lifestyle composites for sleep recovery, activity volume, BMI risk,
  and stress/sleep pressure;
- balanced LightGBM plus balanced XGBoost;
- OOF probability blend sweep instead of a hard-coded blend weight;
- one notebook-generated `submission.csv`.

Promotion gate:

- public score must beat the current `0.94959` champion;
- `fit` and `unhealthy` prediction shares should not collapse;
- local balanced accuracy or macro F1 should stay competitive;
- blend diagnostics should explain the selected LGBM/XGB weight.

### 2. Targeted Interaction Features

Add simple row-safe features before model tuning:

- `sleep_activity_score`: sleep duration plus activity/step/exercise signals;
- `stress_sleep_risk`: high stress plus low sleep or poor sleep quality;
- `activity_intensity`: step count per exercise minute;
- `calorie_per_step`;
- `bmi_sleep_interaction`;
- missing-count feature and selected missing flags.

These are plausible because the EDA shows the target is driven by lifestyle
combinations, not just isolated variables.

### 3. Threshold / Prior Calibration Around Balanced Predictions

Instead of switching to unweighted models, adjust balanced model predictions
more gently:

- start from balanced model probabilities;
- test class multipliers for `fit` and `unhealthy`;
- tune on OOF balanced behavior and prediction mix;
- avoid collapsing minority predictions to train prevalence.

This is safer than the unweighted direction because it keeps the model's useful
minority sensitivity.

## What Not To Do

- Do not submit more raw accuracy-oriented unweighted variants.
- Do not trust local accuracy as the main selector.
- Do not create many public notebooks for small changes.
- Do not add complex feature piles without prediction-mix diagnostics.

## Next Concrete Move

Keep the existing public baseline notebook as the main notebook and make only
small, auditable refinements around the **balanced LGBM/XGB domain ensemble**.
Do not create a new public notebook unless the baseline notebook becomes too
large or slow.

## Implementation Status

The public baseline notebook v8 completed and was submitted:

- row-safe lifestyle interaction features;
- balanced HGB with engineered features;
- balanced LGBM/XGB probability ensemble;
- OOF blend-weight sweep by balanced accuracy;
- champion selection by balanced accuracy and macro F1;
- one notebook-generated `submission.csv`.

Result: public score `0.94959`, replacing `hgb_balanced` (`0.90603`) as the
current champion.

## V10 Calibration Sweep Review

Notebook v10 tested a small class-prior calibration sweep on top of the v8
LGBM/XGB probability blend.

| Candidate | Accuracy | Balanced Accuracy | Macro F1 | Test `at-risk` | Test `fit` | Test `unhealthy` |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| `calibrated_lgbm_xgb_domain_ensemble` | `0.93726` | **`0.94970`** | `0.86257` | `80.92%` | `7.40%` | `11.67%` |
| `lgbm_xgb_domain_ensemble` | `0.93746` | `0.94968` | `0.86308` | `80.94%` | `7.37%` | `11.69%` |
| `hgb_balanced_domain` | **`0.93874`** | `0.94932` | **`0.86510`** | `81.06%` | `7.36%` | `11.58%` |

Best calibration setting:

- `fit_multiplier = 1.12`
- `unhealthy_multiplier = 1.00`

Decision: **do not submit**. The balanced-accuracy gain over v8 is only
`+0.000014`, while accuracy and macro F1 both move down. This is too small to
spend a leaderboard submission under the current quota strategy.
