# Data pipeline maintenance

Goal: the pipeline is reproducible, runs end to end, and its outputs match what downstream code expects.

## Principles

- Raw data is immutable. Stages go `raw → interim → processed → features`, each written to its own location.
- Every stage is a function or script with explicit inputs/outputs and config (no hard-coded paths; paths come from `configs/`).
- Stages are deterministic given config + seed. Record the seed.
- The pipeline has one entry point (e.g. `make data`, `python -m <pkg>.pipeline`, `dvc repro`). Use whatever the project's `CLAUDE.md` names.
- Schemas are written down (pydantic/pandera/pandas dtypes or a YAML schema), not only implied by code.

## Health check procedure

1. **Locate** the pipeline: entry point, stage list, configs, where outputs land. Use `graphify query "data pipeline stages"` first if a graph exists.
2. **Freshness**: compare modification times / hashes of raw inputs vs derived outputs. Flag outputs older than their inputs or code.
3. **Run** the pipeline on a small sample only (`--limit`, `--dry-run`, or the smoke command in `CLAUDE.md` → "Compute budget"). A full pipeline run needs the user's explicit go-ahead. If no cheap mode exists, skip the run, check freshness and schemas on existing outputs instead, and propose adding a `--limit` option.
4. **Validate** each stage output against its schema: columns, dtypes, row counts, null rates, value ranges, uniqueness of IDs.
5. **Drift**: compare summary stats (row count, null %, mean/std of numeric columns, category frequencies) with the previous run if recorded. Flag changes > ~10% or new/missing categories.
6. **Contract with consumers**: check that training/eval code reads the columns the pipeline actually produces.
7. Record the results in the checkpoint report. If stats are not stored anywhere, propose writing them to `reports/data_stats/<date>.json` each run.

## Common fixes you may make on a branch

- Replace hard-coded paths with config.
- Add a schema check at a stage boundary.
- Add a `--limit N` option to make a fast sample run possible.
- Add a unit test for a transform that has none.

Ask before: changing a transform's logic, dropping columns/rows, changing splits, or adding a heavy dependency (DVC, Airflow, etc.).
