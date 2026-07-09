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
4. Log the score immediately in `docs/6_submission_manifest.md`.
5. Promote, revise, or reject the hypothesis.
