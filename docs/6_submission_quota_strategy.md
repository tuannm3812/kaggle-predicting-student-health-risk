# Submission Quota Strategy

Treat notebook runs, local artifact generation, and leaderboard submissions as
separate steps.

## Submit Only When

1. The candidate is reproducible from a notebook.
2. The candidate has a clear hypothesis.
3. OOF validation supports the direction.
4. Prediction distribution looks plausible.
5. The current champion remains available as fallback.

## Do Not Submit

- Blind threshold sweeps.
- Many tiny variants from the same hypothesis.
- Candidates that improve only one fold while damaging class balance.
- Any file whose target column or ID format has not been validated against
  `sample_submission.csv`.

## Practical Rhythm

1. Run notebook and save diagnostics.
2. Compare candidate vs champion.
3. Submit at most one candidate per hypothesis.
4. Log the score immediately in `docs/5_submission_manifest.md`.
5. Promote, revise, or reject the hypothesis.

## Final Champion Gate

**Status: closed.** Final public champion: `0.94959` from the v8 balanced
LGBM/XGB domain-feature ensemble. This gate stayed in force through v23,
the last planned experiment; none of v10-v23 cleared it (`docs/7`), so the
project closed with v8 as champion rather than promoting a new one.

For small calibration or threshold-style changes, do not submit unless the OOF
evidence is stronger than a rounding-level improvement. A candidate should
preferably improve balanced accuracy by at least `0.0002` without reducing macro
F1 or creating a suspicious prediction mix. The v10 class-prior calibration
improved balanced accuracy by only `0.000014` and was therefore not submitted;
the closest any candidate came was v23 at `+0.000111`, still short of the gate.

## Notebook-First Submission Rule

For this project, submissions should come from public Kaggle notebook outputs
whenever possible:

1. Push the notebook with `is_private: false`.
2. Let Kaggle run it to completion.
3. Confirm that `/kaggle/working/submission.csv` is produced.
4. Submit that notebook-generated CSV to the competition.
5. Record the notebook URL, version, CV metrics, prediction mix, and public
   score.

Avoid detached local-only submissions unless the notebook run is blocked.
