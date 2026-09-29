# Sanity checks

Goal: catch the silent bugs that make ML results wrong without anything crashing. Run the checks relevant to what changed; in a checkpoint, run all cheap ones.

Prefer turning each check into a pytest test (`tests/sanity/`) so CI runs it too. When a check is only done by hand, suggest converting it.

## Data

| Check | How |
|---|---|
| Schema | Columns, dtypes, and ranges match the documented schema |
| Nulls / duplicates | Null rate per column and duplicate IDs/rows vs previous run |
| Label distribution | Class balance (or target mean/std) per split; flag big shifts between splits |
| **Leakage: split overlap** | No shared IDs (or near-duplicate rows) between train/val/test |
| **Leakage: time** | For temporal data, all val/test timestamps come after train |
| **Leakage: features** | No feature is a transform of the target or only known after the event; flag any single feature with suspiciously high correlation to the target |
| Group leakage | Same patient/user/session doesn't appear in several splits when it shouldn't |
| Preprocessing fit | Scalers/encoders/imputers are fit on train only |

## Model and training

| Check | How |
|---|---|
| Shapes | Assert input/output shapes at model boundaries |
| Initial loss | Loss at init ≈ expected (e.g. `ln(num_classes)` for balanced CE) |
| Overfit one batch | Model can drive loss near zero on a single small batch; if not, something is broken |
| Loss decreases | Loss goes down over the first steps on real data |
| Baseline | Metric beats a trivial baseline (majority class, mean predictor, linear model) |
| Too good to be true | Near-perfect metrics are treated as a leakage bug until proven otherwise |
| Determinism | Two runs with the same seed give the same metrics (within tolerance) |
| Gradient health | No NaN/Inf; gradient norms not exploding or zero |
| Train/eval mode | Dropout/batchnorm switched correctly; no gradients during eval |

## Evaluation

- Metrics computed on the correct split, never on training data.
- The test set is not used for model selection or tuning; if it was, say so in the report.
- Report variance: several seeds or a confidence interval, not one number.
- Metric implementation checked on a tiny hand-computed example.

## Reporting

Summarise as a table `check | result (pass/warn/fail) | note`. A `fail` on any leakage check goes to the top of the checkpoint summary.
