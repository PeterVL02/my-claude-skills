---
name: ml-workflow
description: Maintain an AI/ML/data-science repo - data pipeline health, leakage and sanity checks, literature research, council direction reviews, a license register and GDPR/ethics checks before publishing, a PROGRESS.md log of experiments and rejected ideas, graphify code graph, git hygiene, tests and CI. Use it in any ML/data repo for setup or a periodic standard or deep "checkpoint", and whenever the user asks whether results can be trusted, whether a model, dataset or output may be used or published, what was tried before, or wants any of these chores done, even without naming the skill.
---

# ML project workflow

This skill keeps an ML/data project healthy and understandable. It has one **setup** mode and several **maintenance** modes that can run alone or together as a **checkpoint**.

Before doing anything, read the project's `CLAUDE.md` and `PROGRESS.md` if they exist. `CLAUDE.md` overrides the defaults here (paths, commands, package manager, data locations). `PROGRESS.md` tells you where the project stands and what has already been tried and rejected.

If `CLAUDE.md` has no ml-workflow sections, the repo hasn't been set up. Single modes (sanity, research, pipeline, ...) still run, but don't create `PROGRESS.md`, the license register or other scaffolding unasked. Offer setup instead. Checkpoints need setup first.

## Hard rules (always apply)

These rules protect things that are expensive or impossible to get back: other people's trust, data, compute, and the project's memory. When a situation isn't covered word for word, reason from the purpose given with each rule.

- **Push only when allowed.** Don't run `git push`, `gh pr create`, `gh pr merge`, or create tags or releases unless the user asked for that specific action in this session. There is one standing exception: a checkpoint pushes its own branch when auto-push applies (`references/checkpoints.md` → "Pushing checkpoint branches"). You may always *ask* to push, create a branch or open a PR, and do it once the user says yes. Why: a push is visible to others and can't really be undone, while everything local can.
- **Work on a branch**, never directly on `main`/`master`. If you're on main with changes to make, create a branch first (naming in `references/git-and-ci.md`). Why: the user reviews and merges on their own terms, and main stays a known-good state.
- **Never force-push, rewrite published history, or delete branches you didn't create.** Why: other people and other checkpoints may build on them, and the loss is silent.
- **Never modify or delete raw data** (`data/raw/` or equivalent). Pipelines read raw data and write derived data elsewhere. Why: raw data is often the only copy, and every result is only reproducible from it.
- **Never commit data files, model weights, secrets, or `.env` files.** Why: git history keeps them forever, even after deletion. They bloat the repo, and a pushed secret or dataset counts as leaked.
- **Register every license.** Before first using a model, dataset, library, framework, service or copied code, add it to `docs/legal/licenses.md`, in the same commit. An unknown license is `unclear`, never `ok`. See `references/legal-ethics.md`. Why: when the project wants to publish or ship, nobody remembers where a component came from, and one incompatible license can block the release. Auto-push also depends on the register being accurate.
- **Sensitive projects: check and ask before every commit.** If `CLAUDE.md` → "Legal and ethics" marks sensitive data:
  - Run the publish check in `references/legal-ethics.md` before every commit and before any push.
  - Get the user's approval for each commit, unless a standing rule under "Commits" or an instruction in this session says otherwise.
  - If you're unsure about a file, don't commit it.
  Why: once data reaches a remote it's effectively published and can't be taken back, and every commit is one push away from that.
- **Respect the compute budget.** Never start full training runs, hyperparameter sweeps, full-dataset pipeline runs, large downloads, or GPU/cluster jobs unless the user explicitly asks in this session.
  - For checks, use the cheap variants listed under "Compute budget" in `CLAUDE.md` (smoke configs, `--limit`, a few steps, tiny fixtures), or read existing logs and metrics instead of re-running.
  - If no cheap variant exists, skip the check, say so in the report, and propose adding one.
  - Default ceiling for any single command during a checkpoint: about 5 minutes, unless `CLAUDE.md` says otherwise.
  
  Why: a real run can take hours of GPU time or money, can block the machine the user is working on, and may overwrite checkpoints or logs they care about. A smoke run answers "is it broken?" just as well.
- **Keep `PROGRESS.md` current.** After any meaningful work (an experiment run, an idea rejected, a decision made, a research finding, a pipeline change), update it before finishing the task. See `references/progress.md`. Why: it's the project's memory across sessions and people. Whatever isn't written there is lost when the chat ends.
- **Check before retrying.** Before proposing or starting an approach, check `PROGRESS.md` → "Tried and rejected". If it's listed there, don't repeat it unless you say what is different this time. Why: re-running a rejected idea is the most expensive kind of wasted work in ML, and the reason it failed usually still holds.
- **Unattended runs.** A run is unattended when the invocation includes the word `unattended`; the scheduled-task prompt is `/ml-workflow checkpoint unattended`. An unattended run can't ask questions. Wherever a step would need an answer, take the safe path that step describes. If a checkpoint can't safely run at all, print a notice and stop (`references/checkpoints.md` → "When a checkpoint doesn't run"). Why: a wrong guess made with nobody watching is found late, and a skipped run costs almost nothing.

## Modes

Choose the mode from the user's request. With no specific request, run **checkpoint**. First check it's configured: `CLAUDE.md` needs a `## Checkpoints` section and a `## Legal and ethics` section that says clearly yes or no on sensitive data (see `references/checkpoints.md` → "When to configure").

| Mode | When | Details |
|---|---|---|
| `setup` | New repo, or repo missing the standard scaffolding | `references/setup.md` |
| `configure` | Set or change how checkpoints run, or the legal settings | `references/checkpoints.md` |
| `progress` | Update or reconstruct the project log | `references/progress.md` |
| `pipeline` | Changes to data loading/processing, or a pipeline check is requested | `references/pipeline.md` |
| `sanity` | After data or model changes, before trusting results | `references/sanity-checks.md` |
| `research` | Looking for related papers/projects or recent developments | `references/research.md` |
| `council` | Direction, prioritisation, whether research is needed, course corrections | `references/council.md` |
| `legal` | Update the license register and run the full legal/ethics review (same as the deep-checkpoint step) | `references/legal-ethics.md` |
| `graph` | Refresh the code knowledge graph | below |
| `ci` | Tests/CI failing, missing, or out of date | `references/git-and-ci.md` |
| `checkpoint` | Periodic health pass; upgrades to deep when one is due | below |
| `deep-checkpoint` | Less frequent pass that adds code review, simplification, a legal review and planning | below |

Load only the reference files for the mode(s) you run.

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

**Precedence:** the hard rules above win over any companion skill. In particular, `git-workflow-and-versioning` assumes trunk-based development. In these projects, work still goes on short-lived branches, and pushing still follows the push rule.

### graph

Uses the graphify skill/CLI (https://github.com/Graphify-Labs/graphify).

1. If `graphify` is not installed, tell the user and suggest `uv tool install graphifyy` or `pipx install graphifyy` (the PyPI name is `graphifyy`). Don't install global tools without asking.
2. If `graphify-out/` doesn't exist, run `/graphify .` (or `graphify .`), then `graphify claude install` so the CLAUDE.md section and search hook exist.
3. If it exists, rebuild/update it with the graphify skill. Confirm the timestamp of `graphify-out/GRAPH_REPORT.md` has changed.
4. Prefer `graphify query "<question>"` for codebase questions before grepping raw files.
5. Ask the user once whether `graphify-out/` should be committed or gitignored, and record the answer in `CLAUDE.md`.

### checkpoint and deep-checkpoint

There are two kinds of checkpoint. A **standard** checkpoint is a frequent, cheap health pass. A **deep** checkpoint also reviews, simplifies, checks legal and ethics, and plans. The project's `CLAUDE.md` → "Checkpoints" decides the cadence, which steps each kind runs, whether deep may change code, whether branches are pushed, and any context (deadlines etc.) that overrides the defaults. See `references/checkpoints.md` for how to read it.

A `checkpoint` run becomes a deep one when a deep checkpoint is due by that config. `deep-checkpoint` always runs deep.

Run these steps in order, carrying findings forward. Skip any step the config turns off for this kind.

1. **Orient.**
   - Read `PROGRESS.md` and `CLAUDE.md` → "Checkpoints" and "Legal and ethics".
   - Find earlier checkpoints, including those on unmerged branches (`references/checkpoints.md` → "Finding earlier checkpoints").
   - Run `git status`, and `git log --since="<last checkpoint date>"` to find what changed. If there was no earlier checkpoint, use the last 7 days.
   - Decide standard or deep.
   - Then check the stop conditions in `references/checkpoints.md` → "When a checkpoint doesn't run": open blocking findings, uncommitted changes, legal settings not configured, or a sensitive project where you may not commit. Unattended, if one applies: print the notice and stop, creating no branch and no report. Interactive: show the notice and ask how to proceed.
2. **Branch.** Note the branch the user is on. Create `chore/checkpoint-YYYY-MM-DD` (deep: `chore/deep-checkpoint-YYYY-MM-DD`) from the current default branch.
3. **graph**: refresh the knowledge graph.
4. **ci**: run the test suite and linters locally. Note failures, fix trivial ones (formatting, imports), and report the rest.
5. **pipeline**: check pipeline health and freshness.
6. **sanity**: run the data and model sanity checks relevant to what changed. Use only smoke-scale runs and existing logs (see the compute budget). Never launch a full training run as part of a checkpoint.
7. **review** (deep): run `/agent-skills:review` on everything changed since the last deep checkpoint (or the last 30 days if there was none). Record Critical and Important findings. Critical findings go under "Blocking findings" in the report.
   - With `report-only` (the default), fix nothing.
   - With `apply-safe`, fix the findings that qualify as small fixes (`references/checkpoints.md` → "Small fixes"), one at a time, each with its own `fix: …` commit. Never auto-fix a Critical finding.
8. **simplify** (deep): run `/agent-skills:code-simplify` on the same scope, then `ponytail-audit` for the whole repo and `ponytail-debt` for the shortcut ledger.
   - With `report-only` (the default), ask code-simplify for findings only and change nothing.
   - With `apply-safe`, let code-simplify apply behaviour-preserving changes one at a time. Run the tests after each and revert any that fail. Commit them separately as `refactor: …`. Never auto-apply `ponytail-audit` deletions; they are judgement calls.
9. **legal** (deep): run the review in `references/legal-ethics.md` → "Deep-checkpoint review". It covers the license register, compatibility with intended use, personal data, whether outputs can identify people, repo history, and ethics. Critical findings go under "Blocking findings". Add missing register rows. Change nothing else, even with `apply-safe`.
10. **research**: one focused search round on the current problem (see the reference for scope). Append to `docs/research/log.md`.
11. **council**: give the council the evidence from the earlier steps plus `PROGRESS.md`, and ask the standard checkpoint questions in `references/council.md`.
12. **plan** (deep): run `/agent-skills:plan` on the council's priorities plus the review, simplify and legal findings. Write the plan to `tasks/plan.md` and `tasks/todo.md`, headed "Proposed by deep checkpoint YYYY-MM-DD, not approved".
    - If those files hold an unfinished plan, write to `docs/checkpoints/YYYY-MM-DD-plan.md` instead.
    - Unattended, skip plan mode and its approval prompt. The plan waits for the user, who runs `/agent-skills:build` once they approve it.
13. If the project's `CLAUDE.md` lists skills under "Checkpoint extras" (for example a diagram refresh), run them now.
14. **progress**: update `PROGRESS.md` with everything learned in this checkpoint, including the council verdict and any ideas it recommended dropping.
15. **Report, commit, push.**
    - Write `docs/checkpoints/YYYY-MM-DD.md` (deep: `YYYY-MM-DD-deep.md`) using the report format below, and commit.
    - If auto-push applies (`references/checkpoints.md` → "Pushing checkpoint branches"), push the branch. Otherwise ask (interactive), or state in the report (unattended), whether to push. Opening a PR always needs the user's go-ahead.
    - Finally, switch back to the branch the user was on.

A checkpoint recommends; it does not act. Don't implement suggestions from the council, the review or the plan during a checkpoint. They go into "Suggested next steps" for the user to approve. The only changes a checkpoint makes are:
- trivial CI fixes (step 4)
- license register rows (step 9)
- what `apply-safe` allows
- the report and `PROGRESS.md`

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
## Remote actions        (branch pushed automatically, or why auto-push didn't apply and what awaits approval)
```

Keep it scannable. Lead with problems, not with what went fine.

**Example blocking findings.** Each item is one checkbox. It gives the evidence (`file:line` or a command) and says why it matters and what would resolve it, so the user can act without rereading the whole report:

```markdown
## Blocking findings
- [ ] B1: Test speakers leak into training — `src/tts/data/split.py:42` — split is by utterance, not speaker, so 38 of 40 test speakers also appear in train; MOS/SECS on test are inflated. Fix: split by `speaker_id` and re-run eval (needs approval: full eval).
- [ ] B2 (from 2026-09-01): License conflict — `docs/legal/licenses.md` row "XYZ-voices" — CC BY-NC 4.0, but CLAUDE.md intended use is commercial. Fix: replace the dataset, get a commercial license, or change the intended use (ask supervisor).
- [x] B3: Speaker IDs in committed fixture — `tests/fixtures/meta.csv` — resolved in a1b2c3d (fixture regenerated synthetically).
```
