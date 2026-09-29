---
name: ml-workflow
description: Maintain an AI/ML/data-science project - data pipelines, sanity checks, research, council direction reviews, license register and legal/ethics (GDPR) checks, a PROGRESS.md project log, graphify code graph, git hygiene, tests and CI. Use in ML/data repos for setup, a periodic standard or deep "checkpoint" (cadence configured per project), or any one of these chores.
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
- **Register every license.** Before first using a model, dataset, library, framework, service or copied code, add it to `docs/legal/licenses.md`, in the same commit. Unknown license = `unclear`, never `ok`. See `references/legal-ethics.md`.
- **Sensitive projects: check before every commit and push.** If `CLAUDE.md` → "Legal and ethics" marks sensitive data, read the staged diff before each commit, and everything a push would publish before asking to push, following `references/legal-ethics.md` → "Publish check". If you're unsure about a file, don't commit it: ask, or hold it back and report it.
- **Sensitive projects: every commit needs the user's approval.** Show the staged files and the publish-check result, then wait for a yes. The exception is when the user has explicitly allowed commits without asking, either in this session or as a standing rule in `CLAUDE.md` → "Legal and ethics" → "Commits". `.claude/settings.json` enforces this with an `ask` rule on `git commit`. Unattended, with no standing rule: don't commit. Leave the work staged on the checkpoint branch and say so in the report.
- **Respect the compute budget.** Never start full training runs, hyperparameter sweeps, full-dataset pipeline runs, large downloads, or GPU/cluster jobs unless the user explicitly asks in this session. For checks, use the cheap variants listed under "Compute budget" in `CLAUDE.md` (smoke configs, `--limit`, a few steps, tiny fixtures), or read existing logs and metrics instead of re-running. If no cheap variant exists, skip that check, say so in the report, and propose adding one. Default ceiling for any single command during a checkpoint: about 5 minutes, unless `CLAUDE.md` says otherwise.
- **Keep `PROGRESS.md` current.** After any meaningful work (experiment run, idea rejected, decision made, research finding, pipeline change), update it before finishing the task. See `references/progress.md`.
- **Check before retrying.** Before proposing or starting an approach, check `PROGRESS.md` → "Tried and rejected". If it is listed, don't repeat it unless you state what is different this time.
- When unattended (scheduled run), do the work locally, commit to a branch, write the report, and stop. Do not push; list what you would push in the report.

## Modes

Choose the mode from the user's request. With no specific request, or when invoked as a scheduled job, run **checkpoint**. First, if `CLAUDE.md` has no `## Checkpoints` section, configure it (see `references/checkpoints.md` → "When to configure").

| Mode | When | Details |
|---|---|---|
| `setup` | New repo or repo missing the standard scaffolding | below |
| `configure` | Set or change when, how often, and how checkpoints run | `references/checkpoints.md` |
| `progress` | Update or reconstruct the project log | `references/progress.md` |
| `pipeline` | Changes to data loading/processing, or pipeline check requested | `references/pipeline.md` |
| `sanity` | After data or model changes, before trusting results | `references/sanity-checks.md` |
| `research` | Looking for related papers/projects or recent developments | `references/research.md` |
| `council` | Direction, prioritisation, whether research is needed, course corrections | `references/council.md` |
| `legal` | License register, publish check, legal/ethics review | `references/legal-ethics.md` |
| `graph` | Refresh the code knowledge graph | below |
| `ci` | Tests/CI failing, missing, or out of date | `references/git-and-ci.md` |
| `checkpoint` | Periodic health pass; upgrades to deep when one is due | below |
| `deep-checkpoint` | Less frequent pass that adds code review, simplification and planning | below |

Load only the reference file for the mode(s) you run.

## Companion skills

These are separate skills that may be installed: the `agent-skills` plugin (addyosmani/agent-skills), the `ponytail` plugin, and the council skill. Use them whenever the situation matches. If one is missing, follow this skill's own reference instead, and don't mention the gap unless it matters. In Claude Code, plugin skills and commands may be namespaced (e.g. `agent-skills:plan`, `ponytail:ponytail`).

**Writing code → ponytail.** Whenever any mode writes or changes code, load `ponytail` first. That covers ci fixes, pipeline code, tests, setup scaffolding, applied simplifications, and implementing approved tasks. Don't load it for research, council, progress or report writing. The hard rules and the project's `CLAUDE.md` conventions win where they clash with it (e.g. configs in YAML, logged seeds).

| Situation | Companion |
|---|---|
| Direction / priority review | council skill (`llm-council`) — see `references/council.md` |
| Underspecified goal or new project | `interview-me`, then `/agent-skills:spec` |
| Rough idea from research or council | `idea-refine` |
| Breaking an agreed change into tasks | `/agent-skills:plan` |
| Implementing approved tasks | `/agent-skills:build` (one task at a time; `auto` only if the user asks) |
| New logic, transforms, metrics, bug fixes | `/agent-skills:test` |
| Tests fail / behaviour unexpected | `debugging-and-error-recovery` |
| High-stakes result or surprising metric | `doubt-driven-development` (fresh-context adversarial check) |
| Before asking to push / open a PR | `/agent-skills:review`, plus `ponytail-review` on the diff |
| Code grew messy | `/agent-skills:code-simplify` (fallback: built-in `/simplify`) |
| Over-engineering, repo-wide | `ponytail-audit`; deferred shortcuts: `ponytail-debt` |
| Library/framework-specific code | `source-driven-development` |
| Quality thresholds for the repo | `/agent-skills:constraints` |
| CI pipeline changes | `ci-cd-and-automation` |
| Significant design decision | `documentation-and-adrs` (write to `docs/decisions/`) |
| Module boundaries / data contracts | `api-and-interface-design` |
| Serving or deploying a model | `observability-and-instrumentation`, then `/agent-skills:ship` before release |

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
   Then fill `CLAUDE.md` → "Legal and ethics". Ask about intended use, sensitive data, the never-publish list and the contact person. Create `docs/legal/licenses.md` from `templates/licenses.md` and fill it with the direct dependencies and every model, dataset and copied code you can find, leaving Status `unclear` for anything you couldn't verify. If the project has sensitive data:
   - Add the backstop hooks from `references/legal-ethics.md` → "Mechanical backstop" to `.pre-commit-config.yaml`, with the regexes agreed with the user.
   - Add `"Bash(git commit:*)"` to the `ask` list in `.claude/settings.json`, so every commit needs approval.
   - Tell the user that unattended checkpoints will then leave their work uncommitted. Ask whether to record a standing exception under "Commits" (e.g. "checkpoint branches may commit without asking"); if they want one, remove the `ask` rule to match.
   Finally, fill `CLAUDE.md` → "Checkpoints" by running the interview in `references/checkpoints.md`. If the repo has no `CONSTRAINTS.md`, offer `/agent-skills:constraints` to set a quality bar (ask; don't run it unprompted).
3. Create any missing standard folders from `references/git-and-ci.md` (with `.gitkeep` where empty). Do not move existing code without asking.
4. Create `docs/research/log.md`, `docs/checkpoints/`, `docs/decisions/`, and `docs/legal/` if missing.
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

### checkpoint and deep-checkpoint

There are two kinds of checkpoint. A **standard** checkpoint is a frequent, cheap health pass. A **deep** checkpoint also reviews, simplifies and plans. The project's `CLAUDE.md` → "Checkpoints" decides the cadence, which steps each kind runs, whether deep may change code, and any context (deadlines etc.) that overrides the defaults. See `references/checkpoints.md` for how to read it.

A `checkpoint` run becomes a deep one when a deep checkpoint is due by that config. `deep-checkpoint` always runs deep.

Run these steps in order, carrying findings forward. Skip any step the config turns off for this kind.

1. Read `PROGRESS.md` and `CLAUDE.md` → "Checkpoints". Find earlier checkpoints, including those on unmerged checkpoint branches (see `references/checkpoints.md` → "Finding earlier checkpoints"). Then run `git status` and `git log --since="<last checkpoint date>"` to find what changed; if there was no earlier checkpoint, use the last 7 days. Decide standard or deep.
   - **Blocked?** A standard checkpoint doesn't run while the newest deep checkpoint has unresolved Critical findings (see `references/checkpoints.md` → "Blocking"). Unattended, stop here: create no branch or report, and print the blocked notice. Interactive, show the notice and ask whether to run anyway. Deep checkpoints and single modes are never blocked.
2. Create branch `chore/checkpoint-YYYY-MM-DD` (deep: `chore/deep-checkpoint-YYYY-MM-DD`) from the current default branch.
3. **graph**: refresh the knowledge graph.
4. **ci**: run the test suite and linters locally. Note failures, fix trivial ones (formatting, imports), and report the rest.
5. **pipeline**: check pipeline health and freshness.
6. **sanity**: run the data and model sanity checks relevant to what changed. Use only smoke-scale runs and existing logs (see compute budget). Never launch a full training run as part of a checkpoint.
7. **review** (deep): run `/agent-skills:review` on everything changed since the last deep checkpoint (or the last 30 days if there was none). Record Critical and Important findings. Critical findings go under "Blocking findings" in the report.
   - With `report-only` (the default), fix nothing.
   - With `apply-safe`, fix the findings that qualify as small fixes (see `references/checkpoints.md` → "Small fixes"), one at a time, each with its own `fix: …` commit. Never auto-fix a Critical finding.
8. **simplify** (deep): run `/agent-skills:code-simplify` on the same scope, then `ponytail-audit` for the whole repo and `ponytail-debt` for the shortcut ledger.
   - With `report-only` (the default), ask code-simplify for findings only and change nothing.
   - With `apply-safe`, let code-simplify apply behaviour-preserving changes one at a time, running tests after each and reverting any that fail. Commit them separately as `refactor: …`. Never auto-apply `ponytail-audit` deletions; they are judgement calls.
9. **legal** (deep): run the review in `references/legal-ethics.md` → "Deep-checkpoint review", covering the license register, compatibility with intended use, personal data, whether outputs can identify people, repo history, and ethics. Critical findings go under "Blocking findings". Add missing register rows. Change nothing else, even with `apply-safe`.
10. **research**: one focused search round on the current problem (see reference for scope). Append to `docs/research/log.md`.
11. **council**: give the council the evidence from the earlier steps plus `PROGRESS.md`, and ask the standard checkpoint questions in `references/council.md`.
12. **plan** (deep): run `/agent-skills:plan` on the council's priorities plus the review, simplify and legal findings. Write the plan to `tasks/plan.md` and `tasks/todo.md`, headed "Proposed by deep checkpoint YYYY-MM-DD, not approved".
    - If those files hold an unfinished plan, write to `docs/checkpoints/YYYY-MM-DD-plan.md` instead.
    - Unattended, skip plan mode and its approval prompt. The plan waits for the user, who runs `/agent-skills:build` once they approve it.
13. If the project's `CLAUDE.md` lists skills under "Checkpoint extras" (for example a diagram refresh), run them now.
14. **progress**: update `PROGRESS.md` with everything learned in this checkpoint, including the council verdict and any ideas it recommended dropping.
15. Write `docs/checkpoints/YYYY-MM-DD.md` (deep: `YYYY-MM-DD-deep.md`) using the report format below, commit, and stop. Ask (or, unattended, state in the report) whether to push and open a PR.

A checkpoint recommends; it does not act. Don't implement suggestions from the council, the review or the plan during a checkpoint. They go into "Suggested next steps" for the user to approve. The only exceptions are the simplifications and small review fixes that `apply-safe` allows.

### Checkpoint report format

```markdown
# Checkpoint YYYY-MM-DD        (append "(deep)" for a deep checkpoint)

## Summary
<3-5 lines: overall health, the one thing that most needs attention>

## Blocking findings     (deep only; omit if none. Standard checkpoints stay blocked until each item is resolved)
- [ ] B1: <Critical finding (review or legal)> — `file:line` — <why it matters, suggested fix>

## Council verdict      (direction: on track / adjust / pivot; top priorities; research needed?)
## Changes since last checkpoint
## Tests & CI            (pass/fail counts, what was fixed, what still fails)
## Pipeline              (stages run, freshness, schema drift, anything broken)
## Sanity checks         (table: check | result | note)
## Research              (1-5 items, each with link and why it matters here)
## Codebase graph        (rebuilt? notable new god nodes / coupling)
## Code review           (deep: Critical/Important findings with file:line; small fixes applied, with commits)
## Simplification        (deep: code-simplify findings or applied commits; ponytail-audit top cuts; ponytail-debt ledger)
## Legal & ethics        (deep: register rows added / unclear / conflict; data-protection and identifiability findings; "Needs a human answer" questions. Any run: "Held back" files not committed by the publish check)
## Plan                  (deep: link to the proposed plan, 3-5 line summary)
## Suggested next steps  (ordered, concrete; mark which came from the council)
## Pending remote actions (branches/PRs you would push, awaiting approval)
```

Keep it scannable. Lead with problems, not with what went fine.
