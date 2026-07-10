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

## Weighting Comparison Result

The v5 baseline notebook answered the weighting question. `hgb_unweighted`
improved OOF accuracy from the balanced baseline's `0.87850` to `0.96519` and
restored a more realistic prediction mix.

| Candidate | Accuracy | Macro F1 | Test `at-risk` | Test `fit` | Test `unhealthy` |
| --- | ---: | ---: | ---: | ---: | ---: |
| `hgb_unweighted` | `0.96519` | `0.90275` | `88.61%` | `4.77%` | `6.62%` |
| `hgb_light_weight` | `0.96438` | `0.90294` | `87.86%` | `5.01%` | `7.13%` |
| `hgb_balanced` | `0.87850` | `0.76104` | `75.71%` | `10.39%` | `13.90%` |

Leaderboard result: the notebook-generated `hgb_unweighted` submission scored
`0.85038`, below the balanced baseline score of `0.90603`.

Decision: reject raw accuracy-oriented `hgb_unweighted` as a leaderboard
direction. The compact comparison remains useful because it shows that local CV
accuracy is not enough for this competition. Future candidates should preserve
class-balanced behavior and use prediction mix plus minority recall as hard
guardrails.

## Leaderboard Improvement Focus

The EDA points to a lifestyle-interaction problem rather than a pure class-prior
problem. The strongest signals are `stress_level`, `physical_activity_level`,
`sleep_duration`, `step_count`, `exercise_duration`, `bmi`, `sleep_quality`,
and `smoking_alcohol`.

Next best direction: add a **CatBoost balanced** section to the existing public
baseline notebook, using native categorical features and preserving the
notebook-generated submission workflow.

Do not optimize for local accuracy alone. The failed `hgb_unweighted`
submission proves that local accuracy can move in the wrong leaderboard
direction.

## Feature Engineering Direction

The baseline notebook now adds row-safe features before the next public run:

- `missing_count`;
- `activity_intensity`;
- `calorie_per_step`;
- `exercise_per_sleep`;
- `steps_per_sleep`;
- `bmi_sleep_interaction`;
- `activity_sleep_score`;
- sleep, stress, activity, and lifestyle interaction flags.

These features are intentionally simple and interpretable. They encode the EDA
signals without using target leakage or external data.
