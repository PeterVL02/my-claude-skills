# <Project name>

<One paragraph: what this project does and for whom.>

## Problem
- Task: <e.g. binary classification of X from Y>
- Data: <source, size, format, where it lives>
- Primary metric: <metric> (baseline: <value>)
- Research focus: <2-4 topics to track in literature/project searches>

## Workflow
This is an ML/data project. Use the `ml-workflow` skill for pipeline maintenance, sanity checks, research, council reviews, the progress log, codebase graph, tests and CI. For a periodic health pass, run the skill in checkpoint mode. Where installed, use the agent-skills companions the skill lists (e.g. test-driven-development, debugging-and-error-recovery, code-review-and-quality). The rules in this file win over any skill.

### Progress log
- Read `PROGRESS.md` at the start of any substantial task.
- Update it before finishing whenever an experiment finishes, an idea is rejected, a decision is made, research turns up something relevant, or the data/pipeline changes.
- Before proposing an approach, check "Tried and rejected". Don't repeat a rejected idea without saying what is different.
- Rejected ideas always get a reason and a "revisit if".

### Council
Run the council skill (`llm-council`) at every checkpoint and at real decision points (choosing between approaches, after surprising results, when stalled). Its output is advice: record it in `PROGRESS.md` as proposed, and don't act on it without my go-ahead.

### Checkpoint extras
<Optional: other skills to run during a checkpoint, e.g. "Update docs/pipeline.svg with the diagram skill". Delete if unused.>

## Git rules
- NEVER run `git push`, `gh pr create`, `gh pr merge`, or create tags/releases unless I explicitly ask in the current session. You may ask me whether to push, create a branch, or open a PR.
- Never commit to `main`. Work on a branch: `feat/`, `fix/`, `exp/`, `data/`, `docs/`, `chore/`.
- Conventional commit messages (`feat: …`, `fix: …`, `data: …`, `exp: …`).
- Never force-push or rewrite pushed history.
- Use short-lived branches even if a skill suggests trunk-based development.

## Commands
- Install: `<uv sync / pip install -e ".[dev]">`
- Pipeline: `<make data / python -m pkg.pipeline>` (fast sample: `<… --limit 1000>`)
- Train: `<python -m pkg.train --config configs/…>`
- Tests: `pytest -m "not slow"`
- Lint/format: `ruff check . && ruff format .`
- Type-check: `mypy src`

## Data rules
- `data/raw/` is read-only. Never modify or delete it.
- Data, model weights, and `.env` are never committed.
- Splits are defined in `<configs/splits.yaml / src/pkg/data/split.py>`; don't change them without asking.

## Code conventions
- Python <3.11+>, type hints on public functions, docstrings on modules and public APIs.
- Logic lives in `src/<package>/`; notebooks are for exploration only.
- Config via YAML in `configs/`; no hard-coded paths or magic numbers.
- Set and log seeds for anything stochastic.

## Codebase graph
- graphify output lives in `graphify-out/` (<committed / gitignored>).
- Prefer `graphify query "<question>"` before grepping for architecture questions.
