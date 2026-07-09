# Predicting Student Health Risk

Kaggle Playground Series S6E7 project for the competition:
https://www.kaggle.com/competitions/playground-series-s6e7

This repository follows the notebook-first workflow used in the previous
Playground episode repos:

- `docs/` stores competition notes, EDA findings, modeling decisions, and
  submission history.
- `notebooks/` stores executable Kaggle/local notebooks.
- `data/` stores local competition files and is intentionally ignored.
- `predictions/` stores OOF/test prediction matrices and is ignored.
- `scratch/` stores helper scripts and temporary automation and is ignored.

## Current Status

The repo scaffold is ready. Kaggle CLI credentials were restored from
`../kaggle_tuannm3812.json` into `~/.kaggle/kaggle.json` with restricted file
permissions. The CLI can list S6E7 files, but download currently returns
`403 Forbidden`, which usually means the Kaggle account still needs to accept
the competition rules in the browser.

Confirmed competition files:

- `train.csv` (`62.7 MB`)
- `test.csv` (`24.6 MB`)
- `sample_submission.csv` (`4.4 MB`)

After credentials are available, run:

```bash
/Users/tuannm3812/Library/Python/3.9/bin/kaggle competitions download -c playground-series-s6e7 -p data
unzip data/playground-series-s6e7.zip -d data
```

Then start with `notebooks/01_eda.ipynb` and record findings in
`docs/2_eda_insights.md`.

## Planned Modeling Ladder

1. EDA and metric confirmation.
2. Stratified cross-validation baseline.
3. Strong single-model GBDT baseline, likely CatBoost/LightGBM/XGBoost.
4. Probability-quality checks: log loss, calibration, class recall, confusion
   matrix, and rare-risk recall.
5. Small OOF-based ensemble and threshold/risk-rule search.
6. Submission only after a written hypothesis and diff/reproducibility check.

See `docs/5_implementation_plan.md` for the full competition plan.
