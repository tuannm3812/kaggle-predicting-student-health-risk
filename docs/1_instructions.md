# Competition Instructions

## Competition

- Name: Predicting Student Health Risk
- Series: Kaggle Playground Series S6E7
- URL: https://www.kaggle.com/competitions/playground-series-s6e7
- Deadline: **2026-07-31 23:59** (confirmed via `kaggle competitions list -s
  playground-series-s6e7`, not the competition page — see the note under
  "Official Competition Instructions" below on why the page itself isn't a
  usable source here).
- Category: Playground · Reward: Swag · Teams: 2,272 (as of 2026-07-20).
- Public leaderboard top: `0.95300` (as of 2026-07-20, via
  `kaggle competitions leaderboard -s`) — already above the `~0.951` figure
  used throughout `docs/9`; consistent with that doc's own finding that the
  visible top keeps floating upward from shared-submission voting, not
  better models. Re-check before trusting the `~0.951` reference elsewhere.

## Official Competition Instructions

**Not filled in from the competition page itself.** Kaggle competition pages
are fully client-side-rendered (React SPA) — both `WebFetch` and a raw
`curl` only return an empty page shell (confirmed 2026-07-20; the HTML has
no server-rendered content, just a `<title>` and a generic meta
description). The Kaggle CLI/API has no endpoint for the Overview/
Evaluation/Rules prose either — `kaggle competitions list/leaderboard` only
return structured metadata (deadline, team count, scores), which is why
those are the only page-sourced facts above.

If this section matters going forward (e.g. reopening the modeling phase,
or reusing this doc as a template for a future competition), paste the
actual text from these tabs on the competition page here:

- **Overview / Description** — the task framing in the competition's own
  words.
- **Evaluation** — the official metric name and formula, if stated. This
  project inferred `balanced accuracy` indirectly (OOF balanced accuracy
  tracked the public score almost exactly across every submission, e.g.
  v8's `0.94975` OOF → `0.94959` LB, and every top public notebook reviewed
  in `docs/9` optimizes the same metric) — worth confirming against the
  actual page text rather than relying on that inference alone.
- **Submission Format** — the exact required CSV header/shape, if stated
  beyond what's already inferred from `sample_submission.csv` below.
- **Rules** — team limits, submission limits, external-data restrictions,
  and anything relevant to what this project's own guardrails
  (`docs/7_submission_quota_strategy.md`) should actually be checked
  against instead of assumed.

## Kaggle Access Troubleshooting

Reusable diagnosis for the CLI/API friction this project hit, not just a
log of what happened once — useful again for this project or the next one.

**Symptom: `kaggle competitions download` returns `403 Forbidden`, but
`kaggle competitions files` works fine with the same credentials.**

- Cause: valid API credentials are not the same as competition access.
  Kaggle requires the account to explicitly accept the competition's rules
  through the web UI before the API will serve the data files, even though
  read-only metadata calls (like listing files) don't require that
  acceptance.
- Fix: open the competition page in a browser, click "Join Competition" /
  accept the rules, then retry the download. No credential change needed.

**Symptom: `kaggle: command not found`.**

- Cause: the Kaggle CLI installed via `pip install --user` isn't on the
  default `PATH` in this environment.
- Fix: use the full path directly, or add it to `PATH` once for the
  session: `/Users/tuannm3812/Library/Python/3.9/bin/kaggle`. Every command
  in this doc and in `docs/0`'s Kaggle-push guidance already uses the full
  path for this reason.

**Verifying access works**, once both are resolved:

```bash
/Users/tuannm3812/Library/Python/3.9/bin/kaggle competitions files playground-series-s6e7
/Users/tuannm3812/Library/Python/3.9/bin/kaggle competitions download -c playground-series-s6e7 -p data
unzip data/playground-series-s6e7.zip -d data
```

This worked without issue for the whole project after the initial rule
acceptance (v3 through the final v25 verification run) — the notebook-based
Kaggle kernel pushes also depend on the same rule acceptance, so if a
future kernel push starts failing to read competition data, check this
first before assuming a code or credential problem.

## Items Confirmed From Local Data

| Item | Status |
| --- | --- |
| Evaluation metric | Balanced accuracy — inferred, not yet confirmed against the official page text (see above). |
| Target column | `health_condition` |
| Submission column name | `health_condition` (two-column submission: `id`, `health_condition`) |
| Train file | `train.csv`, 62.7 MB, 690,088 rows |
| Test file | `test.csv`, 24.6 MB, 295,753 rows |
| Sample submission | `sample_submission.csv`, 4.4 MB |
| Train row count | 690,088 |
| Test row count | 295,753 |
| Missing-value pattern | 449,496 train missing cells; 192,642 test missing cells |
| Target distribution | `at-risk`: 592,561; `unhealthy`: 57,724; `fit`: 39,803 |
| Competition deadline | `2026-07-31 23:59` — confirmed via API (see "Competition" above). |
| Daily submission quota | Not confirmed against the official page; not needed in practice — only 3 leaderboard submissions were ever made across the whole project (`docs/6`), well under any plausible quota. |

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
