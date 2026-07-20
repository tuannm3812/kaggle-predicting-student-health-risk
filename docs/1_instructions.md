# Competition Instructions

Transcribed directly from the competition's Overview/Data pages
(2026-07-20). Kaggle operational troubleshooting (access errors, CLI PATH
issues, why the page can't be fetched by URL) lives in
`docs/0_coding_standards.md` instead — this doc stays limited to what the
competition itself states.

## Competition

- Name: Predicting Student Health Risk
- Series: Kaggle Playground Series S6E7
- URL: https://www.kaggle.com/competitions/playground-series-s6e7
- Citation: Yao Yan, Walter Reade, Elizabeth Park. *Predicting Student
  Health Risk.* https://kaggle.com/competitions/playground-series-s6e7,
  2026. Kaggle.
- License: Attribution 4.0 International (CC BY 4.0)

## Overview

> Welcome to the 2026 Kaggle Playground Series! We plan to continue in the
> spirit of previous playgrounds, providing interesting and approachable
> datasets for our community to practice their machine learning skills, and
> anticipate a competition each month.
>
> Your Goal: Predicting student health risk.

## Evaluation

> Submissions are evaluated on **balanced accuracy** between the predicted
> class and the observed target.

This is an official confirmation, not an inference — every promotion-gate
decision in `docs/7_leaderboard_improvement_insights.md` optimized the
right metric from the start.

### Submission Format

> For each `id` in the test set, you must predict a label (`at-risk`,
> `unhealthy`, `fit`) for the `health_condition` variable. The file should
> contain a header and have the following format:

```
id,health_condition
690088,at-risk
690089,at-risk
690090,at-risk
```

## Timeline

- Start Date: July 1, 2026
- Entry Deadline: same as Final Submission Deadline
- Team Merger Deadline: same as Final Submission Deadline
- **Final Submission Deadline: July 31, 2026**
- All deadlines are 11:59 PM UTC on the stated day unless otherwise noted;
  organizers reserve the right to update the timeline.

This project's own work ran 2026-07-09 through 2026-07-20 — starting 8 days
into the competition window and closing the modeling phase 11 days before
the deadline, on its own evidence (`docs/7`) rather than against the clock.

## About The Tabular Playground Series

> The goal of the Tabular Playground Series is to provide the Kaggle
> community with a variety of fairly light-weight challenges that can be
> used to learn and sharpen skills in different aspects of machine learning
> and data science. The duration of each competition will generally only
> last a few weeks... The challenges will generally use fairly light-weight
> datasets that are synthetically generated from real-world data, and will
> provide an opportunity to quickly iterate through various model and
> feature engineering ideas, create visualizations, etc.

### Synthetically-Generated Datasets

> Using synthetic data for Playground competitions allows us to strike a
> balance between having real-world data (with named features) and
> ensuring test labels are not publicly available... the state-of-the-art
> is much better now than when we started the Tabular Playground Series six
> years ago, and that goal is to produce datasets that have far fewer
> artifacts.

This directly explains a finding from `docs/7`'s external-research section:
v21's exact/near-duplicate row check came back completely empty (zero
duplicates across 985,841 combined rows). That wasn't a failed search — per
this official statement, newer Playground entries like S6E7 are
deliberately engineered to have fewer of exactly that kind of exploitable
generator artifact than older series entries had.

## Dataset Description

> The dataset for this competition (both train and test) was inspired by
> the **College Student Health Behavior Dataset**. Feature distributions
> are close to, but not exactly the same, as the original.

### Files

- `train.csv` — the training set, with `health_condition` as target
- `test.csv` — the test set, used to predict the category for
  `health_condition`
- `sample_submission.csv` — a sample submission file in the correct format

3 files, 91.71 MB total, CSV.

## Prizes

> 1st/2nd/3rd Place — choice of Kaggle merchandise. Kaggle merchandise is
> only awarded once per person across the series, to encourage broader
> participation from beginners.

## Items Confirmed From Local Data

| Item | Status |
| --- | --- |
| Evaluation metric | Balanced accuracy — official (see Evaluation above). |
| Target column | `health_condition` |
| Submission column name | `health_condition` (two-column submission: `id`, `health_condition`) |
| Train file | `train.csv`, 62.7 MB, 690,088 rows |
| Test file | `test.csv`, 24.6 MB, 295,753 rows |
| Sample submission | `sample_submission.csv`, 4.4 MB |
| Train row count | 690,088 |
| Test row count | 295,753 |
| Missing-value pattern | 449,496 train missing cells; 192,642 test missing cells |
| Target distribution | `at-risk`: 592,561; `unhealthy`: 57,724; `fit`: 39,803 |
| Competition deadline | July 31, 2026, 11:59 PM UTC (official, see Timeline above). |
| Daily submission quota | Not stated on the competition page; not needed in practice — only 3 leaderboard submissions were ever made across the whole project (`docs/5`), well under any plausible quota. |

## Expected Input Files

- `train.csv`
- `test.csv`
- `sample_submission.csv`

## Columns

Target:

- `health_condition`

Features:

- Numeric: `sleep_duration`, `heart_rate`, `bmi`, `calorie_expenditure`,
  `step_count`, `exercise_duration`, `water_intake`
- Categorical: `diet_type`, `stress_level`, `sleep_quality`,
  `physical_activity_level`, `smoking_alcohol`, `gender`
