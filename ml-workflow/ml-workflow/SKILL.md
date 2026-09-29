---
name: ml-workflow
description: Maintain an AI/ML/data-science project - data pipelines, sanity checks, research, council direction reviews, a PROGRESS.md project log, graphify code graph, git hygiene, tests and CI. Use in ML/data repos for setup, a periodic "checkpoint", or any one of these chores.
---

# ML project workflow

This skill keeps an ML/data project healthy and understandable. It has one **setup** mode and several **maintenance** modes that can run alone or together as a **checkpoint**.

Before doing anything, read the project's `CLAUDE.md` and `PROGRESS.md` if they exist. `CLAUDE.md` overrides the defaults here (paths, commands, package manager, data locations). `PROGRESS.md` tells you where the project stands and what has already been tried and rejected.

## Hard rules (always apply)

- **Never push without explicit instruction.** Do not run `git push`, `gh pr create`, `gh pr merge`, or anything that changes a remote unless the user asked for that specific action in this session. You MAY *ask* to push, create a branch, or open a PR, and you may do so once the user says yes.
- Work on a branch, never directly on `main`/`master`. If on main with changes to make, create a branch first (see `references/git-and-ci.md` for naming).
- Never force-push, rewrite published history, or delete branches you did not create.
- Never modify or delete raw data (`data/raw/` or equivalent). Pipelines read raw and write derived data elsewhere.
- Never commit data files, model weights, secrets, or `.env` files.
- **Respect the compute budget.** Never start full training runs, hyperparameter sweeps, full-dataset pipeline runs, large downloads, or GPU/cluster jobs unless the user explicitly asks in this session. For checks, use the cheap variants listed under "Compute budget" in `CLAUDE.md` (smoke configs, `--limit`, a few steps, tiny fixtures), or read existing logs and metrics instead of re-running. If no cheap variant exists, skip that check, say so in the report, and propose adding one. Default ceiling for any single command during a checkpoint: about 5 minutes, unless `CLAUDE.md` says otherwise.
- **Keep `PROGRESS.md` current.** After any meaningful work (experiment run, idea rejected, decision made, research finding, pipeline change), update it before finishing the task. See `references/progress.md`.
- **Check before retrying.** Before proposing or starting an approach, check `PROGRESS.md` → "Tried and rejected". If it is listed, don't repeat it unless you state what is different this time.
- When unattended (scheduled run), do the work locally, commit to a branch, write the report, and stop. Do not push; list what you would push in the report.

## Modes

Choose the mode from the user's request. With no specific request, or when invoked as a scheduled job, run **checkpoint**.

| Mode | When | Details |
|---|---|---|
| `setup` | New repo or repo missing the standard scaffolding | below |
| `progress` | Update or reconstruct the project log | `references/progress.md` |
| `pipeline` | Changes to data loading/processing, or pipeline check requested | `references/pipeline.md` |
| `sanity` | After data or model changes, before trusting results | `references/sanity-checks.md` |
| `research` | Looking for related papers/projects or recent developments | `references/research.md` |
| `council` | Direction, prioritisation, whether research is needed, course corrections | `references/council.md` |
| `graph` | Refresh the code knowledge graph | below |
| `ci` | Tests/CI failing, missing, or out of date | `references/git-and-ci.md` |
| `checkpoint` | Periodic health pass; runs everything above | below |

Load only the reference file for the mode(s) you run.

## Companion skills

These are separate skills that may be installed (the `agent-skills` plugin from addyosmani/agent-skills, and the council skill). Use them when present; if one is missing, follow this skill's own reference instead and don't mention the gap unless it matters. In Claude Code, plugin skills may be namespaced (e.g. `agent-skills:test-driven-development`).

| Situation | Companion skill |
|---|---|
| Direction / priority review | council skill (`llm-council`) — see `references/council.md` |
| Underspecified goal or new project | `interview-me`, then `spec-driven-development` |
| Rough idea from research or council | `idea-refine` |
| Breaking an agreed change into tasks | `planning-and-task-breakdown` |
| Implementing a multi-file change | `incremental-implementation` |
| New logic, transforms, metrics, bug fixes | `test-driven-development` |
| Tests fail / behaviour unexpected | `debugging-and-error-recovery` |
| High-stakes result or surprising metric | `doubt-driven-development` (fresh-context adversarial check) |
| Before asking to push / open a PR | `code-review-and-quality` |
| Code grew messy | `code-simplification` |
| Library/framework-specific code | `source-driven-development` |
| Quality thresholds for the repo | `constraint-driven-development` |
| CI pipeline changes | `ci-cd-and-automation` |
| Significant design decision | `documentation-and-adrs` (write to `docs/decisions/`) |
| Module boundaries / data contracts | `api-and-interface-design` |
| Serving or deploying a model | `observability-and-instrumentation`, `shipping-and-launch` |

**Precedence:** the hard rules above win over any companion skill. In particular, `git-workflow-and-versioning` assumes trunk-based development; in these projects, work still goes on short-lived branches, and pushing still needs explicit permission.

### setup

1. Inspect the repo: language, package manager (uv/poetry/pip/conda), existing layout, existing CI, existing `CLAUDE.md`.
2. Copy templates from this skill's `templates/` folder **without overwriting**. If a target file exists, merge the missing parts and show the user the diff:
   - `templates/CLAUDE.md` → `./CLAUDE.md` (fill in the placeholders from what you found; ask about anything you could not infer)
   - `templates/PROGRESS.md` → `./PROGRESS.md` (see step 5)
   - `templates/settings.json` → `./.claude/settings.json`
   - `templates/ci.yml` → `./.github/workflows/ci.yml`
   - `templates/pre-commit-config.yaml` → `./.pre-commit-config.yaml`
   - `templates/gitignore` → append missing lines to `./.gitignore`
   Then find the expensive entry points (training, sweeps, full pipeline runs, evaluation on the full test set). List them in `CLAUDE.md` → "Compute budget", together with their cheap smoke variants, and add each expensive command to the `ask` list in `.claude/settings.json` (e.g. `"Bash(python -m mypkg.train:*)"`, `"Bash(python scripts/train.py:*)"`), so they always need approval. Show the user the list and ask whether anything is missing.
3. Create any missing standard folders from `references/git-and-ci.md` (with `.gitkeep` where empty). Do not move existing code without asking.
4. Create `docs/research/log.md`, `docs/checkpoints/`, and `docs/decisions/` if missing.
5. Fill `PROGRESS.md`. For an existing project, reconstruct it from git history, README, notebooks, and existing notes (see `references/progress.md` → "Reconstructing"), then ask the user to fill gaps, especially rejected ideas and why.
6. Set up graphify (see **graph**).
7. Run tests and pre-commit once so the user starts from a green state, or report what fails.
8. Commit on a branch `chore/ml-workflow-setup` and ask whether to push / open a PR.

### graph

Uses the graphify skill/CLI (https://github.com/Graphify-Labs/graphify).

1. If `graphify` is not installed, tell the user and suggest `uv tool install graphifyy` or `pipx install graphifyy` (PyPI name is `graphifyy`). Do not install global tools without asking.
2. If `graphify-out/` does not exist: run `/graphify .` (or `graphify .`), then `graphify claude install` so the CLAUDE.md section and search hook exist.
3. If it exists: rebuild/update it with the graphify skill. Confirm `graphify-out/GRAPH_REPORT.md` has changed timestamp.
4. Prefer `graphify query "<question>"` for codebase questions before grepping raw files.
5. Ask the user once whether `graphify-out/` should be committed or gitignored, and record the answer in `CLAUDE.md`.

### checkpoint

A full periodic health pass. Run in this order, carrying findings forward:

1. Read `PROGRESS.md`. Then `git status` / `git log --since="<last checkpoint date>"` to find what changed. The last checkpoint date is the newest file in `docs/checkpoints/`; if none, use the last 7 days.
2. Create branch `chore/checkpoint-YYYY-MM-DD` from the current default branch.
3. **graph**: refresh the knowledge graph.
4. **ci**: run the test suite and linters locally; note failures, fix trivial ones (formatting, imports), report the rest.
5. **pipeline**: check pipeline health and freshness.
6. **sanity**: run the data and model sanity checks relevant to what changed, using only smoke-scale runs and existing logs (see compute budget). Never launch a full training run as part of a checkpoint.
7. **research**: one focused search round on the current problem (see reference for scope). Append to `docs/research/log.md`.
8. **council**: give the council the evidence from steps 1-7 plus `PROGRESS.md`, and ask the standard checkpoint questions in `references/council.md`.
9. If the project's `CLAUDE.md` lists skills under "Checkpoint extras" (for example a diagram refresh), run them now.
10. **progress**: update `PROGRESS.md` with everything learned in this checkpoint, including the council verdict and any ideas it recommended dropping.
11. Write `docs/checkpoints/YYYY-MM-DD.md` using the report format below, commit, and stop. Ask (or, unattended, state in the report) whether to push and open a PR.

The council recommends; it does not act. Do not implement its suggestions during a checkpoint. They go into "Suggested next steps" for the user to approve.

### Checkpoint report format

```markdown
# Checkpoint YYYY-MM-DD

## Summary
<3-5 lines: overall health, the one thing that most needs attention>

## Council verdict      (direction: on track / adjust / pivot; top priorities; research needed?)
## Changes since last checkpoint
## Tests & CI            (pass/fail counts, what was fixed, what still fails)
## Pipeline              (stages run, freshness, schema drift, anything broken)
## Sanity checks         (table: check | result | note)
## Research              (1-5 items, each with link and why it matters here)
## Codebase graph        (rebuilt? notable new god nodes / coupling)
## Suggested next steps  (ordered, concrete; mark which came from the council)
## Pending remote actions (branches/PRs you would push, awaiting approval)
```

Keep it scannable. Lead with problems, not with what went fine.
