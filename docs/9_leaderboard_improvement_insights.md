# Leaderboard Improvement Insights

## Current Position

| Candidate | Public Score | Decision |
| --- | ---: | --- |
| `hgb_balanced` | `0.90603` | Current champion |
| `hgb_unweighted` | `0.85038` | Reject |

The public leaderboard top scores are around `0.951`. We need roughly `+0.045`
over the current champion to become competitive.

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

### 1. CatBoost Balanced Baseline

Use CatBoost because it handles categorical fields natively and can model
interactions between stress, activity, sleep, and lifestyle categories.

Recommended setup:

- native categorical columns;
- `auto_class_weights='Balanced'`;
- 5-fold stratified CV;
- same prediction-mix diagnostics;
- one notebook-generated `submission.csv`.

Promotion gate:

- public score must beat `0.90603`;
- `fit` and `unhealthy` prediction shares should not collapse;
- local balanced accuracy or macro F1 should stay competitive.

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

Extend the existing public baseline notebook with a **CatBoost balanced**
section and compare it against `hgb_balanced`. Keep the notebook public and
submit only the notebook-generated `submission.csv` if the diagnostics look
better than the current champion.
