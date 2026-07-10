# Output Insights and Next Steps

## Notebook Submission Provenance

The first leaderboard result comes from the public Kaggle notebook:

- Notebook: `Student Health Risk - Baseline Modeling`
- URL: https://www.kaggle.com/code/tuannm3812/student-health-risk-baseline-modeling
- Version reviewed: public v3
- Output submitted: notebook-generated `submission.csv`
- Public score: `0.90603`

This is the preferred workflow for this competition: keep the notebook public,
make it generate `submission.csv` in `/kaggle/working`, and submit that notebook
output to the leaderboard. No separate zip-style packaging is needed.

## Validation Summary

| Metric | Value |
| --- | ---: |
| OOF accuracy | `0.8785` |
| OOF balanced accuracy | `0.9091` |
| OOF macro F1 | `0.7610` |
| OOF weighted F1 | `0.8910` |
| Public leaderboard | `0.90603` |

The gap between OOF accuracy and public score is positive, so the first public
probe did not expose a validation collapse. The model is useful as a baseline
anchor.

## Class Behavior

| Class | OOF recall | OOF precision | OOF F1 |
| --- | ---: | ---: | ---: |
| `at-risk` | `0.8701` | `0.9875` | `0.9251` |
| `fit` | `0.9226` | `0.5055` | `0.6531` |
| `unhealthy` | `0.9345` | `0.5657` | `0.7048` |

The class-balanced HGB baseline protects minority-class recall very well, but
it creates many false positives for `fit` and `unhealthy`. This is exactly the
trade-off expected from balanced class weighting on an imbalanced target.

## Prediction Mix

| Split | `at-risk` | `fit` | `unhealthy` |
| --- | ---: | ---: | ---: |
| Train target | `85.87%` | `5.77%` | `8.36%` |
| OOF predictions | `75.65%` | `10.53%` | `13.82%` |
| Test submission | `75.71%` | `10.39%` | `13.90%` |

The OOF and test prediction mixes are closely aligned, which is good. However,
both predict minority classes at almost double their training prevalence. That
suggests the next candidate should reduce the class-weight pressure and recover
majority-class precision.

## Decision

Submit this baseline once, record it, and keep it as the class-balanced anchor.
Do not keep submitting small variants of the same HGB setup.

## Next Experiment

Keep the model comparison inside the public baseline notebook instead of
creating many small public notebooks:

1. Unweighted HGB baseline.
2. Lightly weighted HGB baseline.
3. Fully balanced HGB baseline as the existing anchor.
4. Compare OOF accuracy, macro F1, balanced accuracy, and prediction mix.
5. Generate one champion `submission.csv` from the selected candidate.

After the weighting question is answered, add CatBoost or LightGBM as a new
section in the same notebook, unless the notebook becomes too slow or hard to
read.

Only submit the next candidate if it improves the likely leaderboard direction
without collapsing `fit`/`unhealthy` recall.
