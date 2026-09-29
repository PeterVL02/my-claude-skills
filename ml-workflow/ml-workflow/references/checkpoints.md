# Checkpoint configuration

Each project decides when, how often and how its checkpoints run, because workloads, deadlines and compute differ. The settings live in the project's `CLAUDE.md` under `## Checkpoints`. They depend on the project's `## Legal and ethics` section, which decides whether checkpoint branches may be pushed and whether checkpoints may commit on their own. Templates for both are in `templates/CLAUDE.md`.

## When to configure

- During **setup** (`references/setup.md`, steps 4-5).
- In **configure** mode, e.g. when a deadline approaches, the pace of work changes, or the project gets new data.
- When a checkpoint starts and the configuration is incomplete:
  - **`## Checkpoints` missing.** Interactive: configure first, then run the checkpoint. Unattended: use the defaults below, add the section to `CLAUDE.md` on the checkpoint branch, and flag it at the top of the report so the user reviews it.
  - **`## Legal and ethics` missing, or "Sensitive data" isn't a clear yes or no.** This is common in repos set up before the legal section existed. Interactive: run the interview in `references/legal-ethics.md` → "Configure" first. Unattended: stop with the "legal not configured" notice (see "When a checkpoint doesn't run"). Guessing wrong here could publish data, and only the user can answer.

## Interview

Read the repo, `PROGRESS.md` and git activity first, so each question comes with a suggested answer. Ask the questions below in one round (use AskUserQuestion if available). Offer the default as the first option, and phrase every question so that "I don't know" still gives a working config.

If the legal section is missing, or it's unclear whether the project holds anything sensitive, ask the legal questions first (`references/legal-ethics.md` → "Configure"). Question 5 depends on the answers.

1. **Standard checkpoint: how and how often?** Options include a daily or weekly scheduled task, or manual only. Suggest a cadence based on how active the repo is (commits per week).
2. **Deep checkpoint: how often?** `every N days` (default 30), `every Nth checkpoint`, or `manual`. Suggest shorter intervals for fast-moving or deadline-driven projects.
3. **Extra steps in the standard checkpoint?** (multi-select) research, council, review, simplify, legal. Default: none; they stay in the deep checkpoint only. Suggest legal for projects with sensitive data.
4. **Deep checkpoint code changes?** `report-only` (default) or `apply-safe`. `apply-safe` applies behaviour-preserving simplifications and small review fixes (see "Small fixes"), each as its own commit on the checkpoint branch.
5. **Push checkpoint branches?** `auto` (default) or `never`. Tell the user whether `auto` would push today, and if not, why not (see "Pushing checkpoint branches"). For sensitive projects, `auto` never pushes, so offer only `never`.

Then ask in one plain line whether there's any context that should change how checkpoints run: a deadline, a crunch period, a paused workstream, a compute freeze. Record the answer under "Context", or write "none".

Write the section, show the user the diff, and commit it. If push is `auto`, update `.claude/settings.json` as described in "Pushing checkpoint branches".

If the standard checkpoint is scheduled, remind the user to create the scheduled task themselves, with the prompt `/ml-workflow checkpoint unattended`. The word `unattended` is how the skill knows nobody is there to answer. One task is enough, because standard runs upgrade to deep when one is due. A deep cadence of `manual` means they run `/ml-workflow deep-checkpoint` themselves.

## Defaults

| Setting | Default |
|---|---|
| Standard checkpoint | manual |
| Deep checkpoint | every 30 days |
| Deep code changes | report-only |
| Push checkpoint branches | auto (pushes only when the conditions below hold) |
| Standard steps | graph, ci, pipeline, sanity |
| Deep steps | all |

## Reading the config at run time

- **Is a deep checkpoint due?** Use the earlier checkpoints found as described below.
  - `every N days`: due when the newest deep checkpoint is older than N days, or when there is none.
  - `every Nth checkpoint`: due when N−1 or more standard checkpoints have run since the newest deep one.
  - `manual`: never due.
- **Context overrides the table.** If Context says e.g. "deadline 2026-11-15: skip research", follow it until the date passes. Then note in the report that the override has expired, and suggest removing it.
- **Unknown or contradictory settings:** use the default for that setting and say so in the report. The exception is the legal settings, where you stop instead (see "When to configure").

## Pushing checkpoint branches

Checkpoint branches only hold reports, `PROGRESS.md` updates and small safe fixes. Pushing them lets the user review a checkpoint from anywhere and keeps a backup. That's worth doing automatically only when nothing in the repo could leak.

**Conditions for `auto`.** Re-check them at step 15 of every checkpoint, because they can change between runs. Push only if all of these hold:

- `CLAUDE.md` → "Legal and ethics" says the project has **no** sensitive data. If it says yes, or is missing, or is unclear, don't push.
- In `docs/legal/licenses.md`, every **model** and **dataset** row has Status `ok` and a permissive license. Permissive means it allows use, modification and redistribution with at most attribution: e.g. MIT, Apache-2.0, BSD, CC0, CC BY, ODC-By, PDDL, public domain. These don't qualify: non-commercial, no-derivatives, share-alike, research-only, use-restricted (RAIL/OpenRAIL, community licenses) and unknown licenses.
- No `conflict` row anywhere in the register.
- No open legal item under "Blocking findings" in the newest deep checkpoint.
- The standard publish check on the branch finds nothing: no data files, weights, secrets or identifiers in `git log -p <default-branch>..HEAD`.

**When they hold:**
- Run `git push -u origin <checkpoint-branch>`, in exactly that form, so it matches the allow rule below.
- Record the push under "Remote actions" in the report.
- Never push anything other than the checkpoint's own branch.
- Never open a PR without asking.

**When they don't:** don't push. Under "Remote actions", say which condition failed, and that the branch awaits approval.

**Permissions.** An `ask` rule always wins over an `allow` rule in `.claude/settings.json`. So the template's `"Bash(git push:*)"` ask rule would stop even an allowed push. With push `auto`:
- Remove `"Bash(git push:*)"` from `ask`.
- Add `"Bash(git push -u origin chore/checkpoint-*)"` and `"Bash(git push -u origin chore/deep-checkpoint-*)"` to `allow`.
- The template's `deny` rules still block force-pushes and pushes to `main`/`master`.
- Other pushes then fall back to the session's permission mode, which prompts in the default mode.
- The push rule in SKILL.md still applies to every other branch.

With push `never`, or when the project becomes sensitive, put the ask rule back and remove the allow rules.

## When a checkpoint doesn't run

Check all of these at step 1. If any applies, the run doesn't go ahead:
- **Unattended:** print the notice and stop. Don't create a branch or a report, and change nothing.
- **Interactive:** show the notice and ask how to proceed.

Each notice names the problem, lists the relevant files or findings, and gives the ways to fix it.

| Condition | Applies to | Interactive options | Notice tells the user to |
|---|---|---|---|
| **Blocked:** open blocking findings (see "Blocking") | standard only | run anyway (write "ran while blocked (user override)" at the top of the Summary; the next run is blocked again) | fix and merge, tick the item, say it's accepted/deferred, or run `/ml-workflow deep-checkpoint` |
| **Uncommitted changes:** `git status` isn't clean | all | commit them first, stash them, or cancel | commit or stash; checkpoints won't switch branches over someone's work in progress |
| **Legal not configured:** see "When to configure" | all | run the legal interview now | run `/ml-workflow configure` |
| **Sensitive, may not commit:** sensitive data, and no standing "Commits" rule that lets checkpoint branches commit | all | approve each commit as it comes | run it interactively, or add a "Commits" exception in `CLAUDE.md` |

Single modes are never stopped by these conditions. They still follow the hard rules.

**Example notice** (unattended; several conditions can apply at once, so list them all). Print it as the run's only output, so the scheduled task's log shows it at a glance:

```text
ml-workflow: checkpoint 2026-10-02 did not run. Nothing was changed.

Blocked by 1 open finding in the newest deep checkpoint
(branch chore/deep-checkpoint-2026-09-28, docs/checkpoints/2026-09-28-deep.md):
  - B1: Test speakers leak into training — src/tts/data/split.py:42
Uncommitted changes in the working copy:
  - M src/tts/train.py
  - ?? notebooks/scratch.ipynb

To unblock:
  - fix B1 and merge it, tick it in the report, or tell Claude it's accepted/deferred
    (or run /ml-workflow deep-checkpoint, which re-checks it)
  - commit or stash the changes above
```

## Finding earlier checkpoints

Checkpoints commit to their own branches, and the user may not have merged them yet. So don't rely only on `docs/checkpoints/` on the default branch:

- Look for reports in `docs/checkpoints/` on the default branch.
- Look for branches, both local and on the remote: `git branch -a --list "*chore/checkpoint-*" "*chore/deep-checkpoint-*"`. The branch name gives the date and kind. Read the report with `git show <branch>:docs/checkpoints/<date>.md` (deep: `<date>-deep.md`).
- The same date on several copies means it's the same checkpoint. Read all copies, because the user may have edited any of them.
- If today's branch name already exists, append `-2`, `-3`, … to both the branch and the report name.

## Small fixes (apply-safe)

A review finding may be fixed during a deep checkpoint only if **all** of these hold:

- It is Important or a Suggestion, never Critical.
- The fix is obvious and local: about 10 changed lines at most, in one file, plus a test if needed.
- It can't change results. It doesn't touch data loading, splits, features, metrics or evaluation code, training configs, default hyperparameters or seeds. It also leaves dependencies, public APIs, CI, `.claude/settings.json` and security-sensitive code alone.

Typical small fixes: typos in strings, docs or log messages; a misleading docstring; an unused import; an unclosed file handle; the wrong exception type; an off-by-one in a logging or progress helper. Anything else goes to "Suggested next steps".

For each fix:

1. Load `ponytail`.
2. For a bug, use `/agent-skills:test` (Prove-It: write a failing test first). Then run the test suite.
3. If anything fails, revert the fix and report the finding instead.
4. Commit it on its own as `fix: <what> (review <finding id>)`.

Apply at most 10 fixes per checkpoint so the branch stays easy to review.

## Blocking

Critical review and legal findings from a deep checkpoint block the standard checkpoints that follow, until they're resolved. The idea is that routine health passes shouldn't pile up on top of a known serious problem.

- **What blocks:** every unticked item under "Blocking findings" in the newest deep checkpoint's report. A new deep checkpoint re-checks the previous deep checkpoint's open items and carries the unresolved ones into its own list, keeping their IDs (e.g. `B1 (from 2026-09-30)`). That way the newest deep report is always the complete list.
- **An item is resolved when either:**
  - it's ticked `[x]` in any copy of the report, or
  - the default branch contains the fix. Check this by rereading the code at `file:line` and running any test the finding names. Don't assume: if you're unsure, it's still open. A fix that's only on an unmerged branch doesn't count.
- **Resolving in a session:** when the user says a finding is fixed, accepted or deferred:
  - Tick it in the report on its checkpoint branch and commit there.
  - If that branch is already merged, tick it on a new `docs/resolve-<finding id>` branch, never on the default branch.
  - Also add the decision and its reason to `PROGRESS.md` → "Decisions log" on that branch.
  - Checking out an existing local branch and committing to it is allowed. Pushing follows the push rule.
