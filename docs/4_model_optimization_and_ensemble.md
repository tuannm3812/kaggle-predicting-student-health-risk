# Model Optimization and Ensemble

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

Write lightweight diagnostics:

- `oof_predictions.csv` or model-specific `.npy` files.
- `feature_results.csv`.
- `model_results.csv`.
- `blend_results.csv`.
- `submission.csv`.

Keep raw prediction artifacts ignored by git.
