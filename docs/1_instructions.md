# Competition Instructions

## Competition

- Name: Predicting Student Health Risk
- Series: Kaggle Playground Series S6E7
- URL: https://www.kaggle.com/competitions/playground-series-s6e7

## Local Access Status

Kaggle API package is installed. Credentials were restored from
`../kaggle_tuannm3812.json` into `~/.kaggle/kaggle.json` and permissions were
set to `600`.

The CLI can list competition files, but download currently returns
`403 Forbidden`. This usually means the account must accept the competition
rules on Kaggle before API download is allowed.

Once credentials are restored, run:

```bash
/Users/tuannm3812/Library/Python/3.9/bin/kaggle competitions files playground-series-s6e7
/Users/tuannm3812/Library/Python/3.9/bin/kaggle competitions download -c playground-series-s6e7 -p data
unzip data/playground-series-s6e7.zip -d data
```

## Items To Confirm From Data Page

Fill these in immediately after data access:

| Item | Status |
| --- | --- |
| Evaluation metric | To confirm |
| Target column | `health_condition` |
| Submission column name | To confirm |
| Train file | `train.csv`, 62.7 MB, 690,088 rows |
| Test file | `test.csv`, 24.6 MB, 295,753 rows |
| Sample submission | `sample_submission.csv`, 4.4 MB |
| Train row count | 690,088 |
| Test row count | 295,753 |
| Missing-value pattern | 449,496 train missing cells; 192,642 test missing cells |
| Target distribution | `at-risk`: 592,561; `unhealthy`: 57,724; `fit`: 39,803 |
| Competition deadline | To confirm |
| Daily submission quota | To confirm |

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
