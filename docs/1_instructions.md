# Competition Instructions

## Competition

- Name: Predicting Student Health Risk
- Series: Kaggle Playground Series S6E7
- URL: https://www.kaggle.com/competitions/playground-series-s6e7

## Local Access Status

Resolved. The initial `403 Forbidden` on download meant the account hadn't
yet accepted the competition rules on Kaggle; once accepted, both the local
CLI and the two public Kaggle kernels' notebook-generated downloads worked
without issue for the whole project (v3 through the final v25 verification
run). Kaggle CLI path: `/Users/tuannm3812/Library/Python/3.9/bin/kaggle`
(not on default `PATH`). To refresh local data:

```bash
/Users/tuannm3812/Library/Python/3.9/bin/kaggle competitions files playground-series-s6e7
/Users/tuannm3812/Library/Python/3.9/bin/kaggle competitions download -c playground-series-s6e7 -p data
unzip data/playground-series-s6e7.zip -d data
```

## Items To Confirm From Data Page

| Item | Status |
| --- | --- |
| Evaluation metric | Balanced accuracy — inferred, not stated on the competition page, but confirmed indirectly: OOF balanced accuracy tracked the public score almost exactly across every submission (e.g. v8's `0.94975` OOF → `0.94959` LB), and every top public notebook reviewed (`docs/9`) optimizes the same metric. |
| Target column | `health_condition` |
| Submission column name | `health_condition` (two-column submission: `id`, `health_condition`) |
| Train file | `train.csv`, 62.7 MB, 690,088 rows |
| Test file | `test.csv`, 24.6 MB, 295,753 rows |
| Sample submission | `sample_submission.csv`, 4.4 MB |
| Train row count | 690,088 |
| Test row count | 295,753 |
| Missing-value pattern | 449,496 train missing cells; 192,642 test missing cells |
| Target distribution | `at-risk`: 592,561; `unhealthy`: 57,724; `fit`: 39,803 |
| Competition deadline | Not tracked — the modeling phase closed on its own evidence (`docs/9`), not against a deadline, so this was never needed. |
| Daily submission quota | Not tracked — only 3 leaderboard submissions were ever made across the whole project (`docs/6`), well under any plausible quota. |

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
