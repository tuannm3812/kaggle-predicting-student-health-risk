# Model Optimization and Ensemble

**Status: historical.** This is the original day-1 optimization plan, kept as
a record of starting intent; it was never split into its own notebook. Every
candidate listed below (HP search, feature-family validation, calibration,
threshold adjustment, OOF blending) was actually tried as a config-flagged
section inside `02_baseline_modeling.ipynb` — see `docs/9_leaderboard_improvement_insights.md`
for the full experiment ledger (v10–v23) and `README.md` for the closing
summary. The "Artifact Policy" filenames below are the planned names, not the
ones the notebook actually writes — see the corrected list at the end of this
doc.

## Objective

Use a compact, evidence-driven optimization path similar to the strongest prior
episode repos: tune the best single models, save OOF predictions, then ensemble
only when model diversity is visible in validation.

## Candidate Optimizations

1. Focused hyperparameter search for LightGBM, CatBoost, and XGBoost.
2. Feature-family validation: raw, categorical-aware, interaction, ratio, and
   threshold features.
3. Probability calibration if the metric or submission format rewards
   probability quality.
4. Threshold or class-prior adjustment if the metric is macro F1, balanced
   accuracy, or rare-class recall sensitive.
5. Small OOF blend grid or logistic meta-learner over saved model probabilities.

## Anti-Overfit Rules

- No blind leaderboard sweeps.
- No target encoding outside folds.
- No large ensemble unless it beats the best single model in OOF.
- Every submitted candidate needs a written hypothesis and artifact trace.

## Artifact Policy

Write lightweight diagnostics. Actual filenames written by
`02_baseline_modeling.ipynb`, gitignored under `predictions/`:

- `baseline_model_comparison.csv`
- `blend_weight_results.csv`
- `champion_oof_predictions.csv`
- `target_encoding_blend_results.csv` (v20)
- `native_categorical_blend_results.csv` (v23)
- `submission.csv`

Keep raw prediction artifacts ignored by git.
