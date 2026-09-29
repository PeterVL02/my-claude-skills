# <Project name>

<One paragraph: what this project does and for whom.>

## Problem
- Task: <e.g. binary classification of X from Y>
- Data: <source, size, format, where it lives>
- Primary metric: <metric> (baseline: <value>)
- Research focus: <2-4 topics to track in literature/project searches>

## Workflow
This is an ML/data project. Use the `ml-workflow` skill for pipeline maintenance, sanity checks, research, council reviews, the progress log, codebase graph, tests and CI. For a periodic health pass, run the skill in checkpoint mode. Where installed, use the agent-skills companions the skill lists (e.g. `/agent-skills:test`, `/agent-skills:review`, `/agent-skills:plan`, debugging-and-error-recovery). Whenever you write or change code, load the `ponytail` skill first. Don't load it for research, writing or planning. The rules in this file win over any skill.

### Progress log
- Read `PROGRESS.md` at the start of any substantial task.
- Update it before finishing whenever an experiment finishes, an idea is rejected, a decision is made, research turns up something relevant, or the data/pipeline changes.
- Before proposing an approach, check "Tried and rejected". Don't repeat a rejected idea without saying what is different.
- Rejected ideas always get a reason and a "revisit if".

### Council
Run the council skill (`llm-council`) at checkpoints that include it (see "Checkpoints") and at real decision points (choosing between approaches, after surprising results, when stalled). Its output is advice: record it in `PROGRESS.md` as proposed, and don't act on it without my go-ahead.

## Checkpoints
Read by the `ml-workflow` skill. Edit here, or run `/ml-workflow configure`.

- Standard checkpoint: <how and how often, e.g. "weekdays 08:00, desktop scheduled task" / "manual">
- Deep checkpoint: <every 30 days / every 4th checkpoint / manual>
- Deep code changes: <report-only / apply-safe (safe simplifications + small review fixes, each its own commit)>
- Push checkpoint branches: <auto (only when there's no sensitive data, all models and datasets have permissive licenses and pass the publish check) / never>
- Scheduled task prompt: `/ml-workflow checkpoint unattended`
- Critical review and legal findings block standard checkpoints until they're fixed, ticked in the deep report, or accepted.
- Context: <deadlines, crunch periods, compute freezes that change how checkpoints run, e.g. "paper deadline 2026-11-15: no research or council until then"; or "none">

| Step | Standard | Deep |
|---|---|---|
| graph | yes | yes |
| ci | yes | yes |
| pipeline | yes | yes |
| sanity | yes | yes |
| research | no | yes |
| council | no | yes |
| review (`/agent-skills:review`) | no | yes |
| simplify (`/agent-skills:code-simplify`, `ponytail-audit`, `ponytail-debt`) | no | yes |
| legal (licenses, data protection, ethics) | no | yes |
| plan (`/agent-skills:plan`) | no | yes |
| extras (below) | yes | yes |

Progress update and the checkpoint report always run.

### Checkpoint extras
<Optional: other skills to run during a checkpoint, e.g. "Update docs/pipeline.svg with the diagram skill". Delete if unused.>

## Git rules
- NEVER run `git push`, `gh pr create`, `gh pr merge`, or create tags/releases unless I explicitly ask in the current session. You may ask me whether to push, create a branch, or open a PR. The only exception: a checkpoint pushes its own branch when "Push checkpoint branches" is `auto` and its conditions hold.
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

## Compute budget
Never run these without my explicit go-ahead in the current session. For checks, use the smoke variant or read existing logs.

| Expensive command | What it does | Approx. cost | Smoke variant |
|---|---|---|---|
| `<python -m pkg.train --config configs/full.yaml>` | Full training | <hours, GPU> | `<python -m pkg.train --config configs/smoke.yaml>` (<1-2 min, CPU) |
| `<make data>` | Full pipeline | <…> | `<make data LIMIT=1000>` |

- Max runtime for any command during a checkpoint: <5 min>.
- Latest real training logs: `<wandb project / mlruns/ / logs/>` — use these for training-behaviour checks.

## Data rules
- `data/raw/` is read-only. Never modify or delete it.
- Data, model weights, and `.env` are never committed.
- Splits are defined in `<configs/splits.yaml / src/pkg/data/split.py>`; don't change them without asking.

## Legal and ethics
Read by the `ml-workflow` skill (`references/legal-ethics.md`). Not legal advice; when unsure, ask the contact below.

- Intended use of outputs: <research only / publication / commercial / open release of weights>. Project license: <MIT / none yet>.
- License register: `docs/legal/licenses.md`. Add a row before using any new model, dataset, library, framework, service or copied code.
- Sensitive data: <no / yes: what it is (e.g. voice recordings of identifiable speakers), under what terms (consent form, data-use agreement), which rules apply (GDPR, …) / yes (unconfirmed: <open question>)>
- Never publish (commit or push): <e.g. anything under data/, audio of training speakers, speaker IDs, models trained on the data, per-speaker metrics; or "standard rules only">
- Commits: <no extra rule (not sensitive) / checkpoint branches may commit locally after the publish check passes; every other commit asks (recommended when sensitive) / every commit asks (scheduled checkpoints won't run)>
- Publishing outputs (weights, samples, demos, papers) requires: <e.g. speaker-anonymity check passed + supervisor sign-off; or "nothing extra">
- Contact for legal/data questions: <supervisor / data owner / DPO>

## Code conventions
- Python <3.11+>, type hints on public functions, docstrings on modules and public APIs.
- Logic lives in `src/<package>/`; notebooks are for exploration only.
- Config via YAML in `configs/`; no hard-coded paths or magic numbers.
- Set and log seeds for anything stochastic.

## Codebase graph
- graphify output lives in `graphify-out/` (<committed / gitignored>).
- Prefer `graphify query "<question>"` before grepping for architecture questions.
