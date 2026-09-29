# Checkpoint configuration

Each project decides when, how often, and how its checkpoints run, because workloads, deadlines and compute differ. The settings live in the project's `CLAUDE.md` under `## Checkpoints` (template: `templates/CLAUDE.md`).

## When to configure

- During **setup** (step 2).
- In **configure** mode, e.g. when a deadline approaches or the pace of work changes.
- When the skill runs with no specific request and `CLAUDE.md` has no `## Checkpoints` section. If you're interactive, configure first and then run the checkpoint. If unattended, use the defaults below, add the section to `CLAUDE.md` on the checkpoint branch, and flag it at the top of the report so the user reviews it.

## Interview

Read the repo, `PROGRESS.md` and git activity first, so each question comes with a suggested answer. Ask the questions below in one round (use AskUserQuestion if available). Offer the default as the first option, and phrase every question so that "I don't know" still gives a working config.

1. **Standard checkpoint: how and how often?** Options include a daily or weekly scheduled task, or manual only. Suggest a cadence from how active the repo is (commits per week).
2. **Deep checkpoint: how often?** `every N days` (default 30), `every Nth checkpoint`, or `manual`. Suggest shorter intervals for fast-moving or deadline-driven projects.
3. **Extra steps in the standard checkpoint?** (multi-select) research, council, review, simplify. Default: none; they stay in the deep checkpoint only.
4. **Deep checkpoint code changes?** `report-only` (default) or `apply-safe`. `apply-safe` applies behaviour-preserving simplifications and small review fixes (see "Small fixes"), each as its own commit on the checkpoint branch.

Then ask in one plain line whether there's any context that should change how checkpoints run: a deadline, a crunch period, a paused workstream, a compute freeze. Record the answer under "Context", or write "none".

Write the section, show the user the diff, and commit it. If the standard checkpoint is scheduled, remind the user to create the scheduled task themselves (prompt: `/ml-workflow checkpoint`). One task is enough, because standard runs upgrade to deep when due. A deep cadence of `manual` means they run `/ml-workflow deep-checkpoint` themselves.

## Defaults

| Setting | Default |
|---|---|
| Standard checkpoint | manual |
| Deep checkpoint | every 30 days |
| Deep code changes | report-only |
| Standard steps | graph, ci, pipeline, sanity |
| Deep steps | all |

## Reading the config at run time

- **Is a deep checkpoint due?** Use the earlier checkpoints found as described below.
  - `every N days`: due when the newest deep checkpoint is older than N days, or when there is none.
  - `every Nth checkpoint`: due when N−1 or more standard checkpoints have run since the newest deep one.
  - `manual`: never due.
- **Context overrides the table.** If Context says e.g. "deadline 2026-11-15: skip research", follow it until the date passes. Then note in the report that the override has expired, and suggest removing it.
- **Unknown or contradictory settings:** use the default for that setting and say so in the report.

## Finding earlier checkpoints

Checkpoints commit to their own branches, and you may not have merged them yet. So don't rely only on `docs/checkpoints/` on the default branch:

- Look for reports in `docs/checkpoints/` on the default branch.
- Look for local branches: `git branch --list "chore/checkpoint-*" "chore/deep-checkpoint-*"`. The branch name gives the date and kind. Read the report with `git show <branch>:docs/checkpoints/<date>.md` (deep: `<date>-deep.md`).
- The same date on both means it's the same checkpoint. Read both copies, because the user may have edited either one.
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

Critical review findings from a deep checkpoint block the standard checkpoints that follow, until they're resolved. The idea is that routine health passes don't pile up on top of a known serious problem.

- **What blocks:** every unticked item under "Blocking findings" in the newest deep checkpoint's report. A new deep checkpoint re-checks the previous deep checkpoint's open items and carries the unresolved ones into its own list, keeping their IDs (e.g. `B1 (from 2026-09-30)`). That way the newest deep report is always the complete list.
- **An item is resolved when either:**
  - it's ticked `[x]` in either copy of the report, or
  - the default branch contains the fix. Check this by rereading the code at `file:line` and running any test the finding names. Don't assume: if you're unsure, it's still open. A fix that's only on an unmerged branch doesn't count.
- **Resolving in a session:** when the user says a finding is fixed, accepted or deferred, tick it in the report on its checkpoint branch and commit there. If that branch is already merged, tick it on a new `docs/resolve-<finding id>` branch, never on the default branch. Also add the decision and its reason to `PROGRESS.md` → "Decisions log" on that branch. Checking out an existing local branch and committing to it is allowed; pushing still isn't.
- **Blocked notice:** give the open items with `file:line`, the report's location (branch or path), and the ways to unblock:
  - fix it and merge the fix into the default branch
  - tick the item in the report
  - tell Claude it's accepted or deferred
  - run `/ml-workflow deep-checkpoint`, which re-checks everything
- **Interactive override:** if the user chooses to run anyway, run the standard checkpoint and write "ran while blocked (user override)" at the top of its Summary. The override doesn't resolve anything; the next run is blocked again.
