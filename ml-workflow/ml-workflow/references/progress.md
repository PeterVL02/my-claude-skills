# PROGRESS.md: the project log

Goal: an outsider (new teammate, supervisor, future you, a fresh Claude session) can read `PROGRESS.md` and understand where the project is, what has been done, what was tried and why it failed, and what comes next, without reading the code or the chat history.

`PROGRESS.md` lives in the repo root and is committed. Template: `templates/PROGRESS.md`.

## How it relates to other files

| File | Role |
|---|---|
| `PROGRESS.md` | The narrative and the index. Short entries, links out for detail. |
| `docs/research/log.md` | Full research notes; PROGRESS.md links to the relevant entries. |
| `docs/checkpoints/*.md` | Point-in-time health reports; PROGRESS.md records what they concluded. |
| `docs/decisions/*.md` | ADRs for big choices; PROGRESS.md's decisions log links to them. |
| `CLAUDE.md` | How to work in the repo (rules, commands). Not a status file. |

Don't duplicate long content. Summarise in one or two lines and link.

## When to update

Update in the same session, before finishing, whenever any of these happen:

- An experiment finishes (even a failed one): what, config/branch, result, conclusion.
- An idea is rejected: move it to "Tried and rejected" with the reason.
- A decision is made (by the user, or proposed by the council).
- A research finding or new lead appears.
- The pipeline, data, splits, or metric definition change.
- The current best result changes.
- A checkpoint runs.

Small edits (typo fixes, refactors with no behaviour change) don't need an entry.

## Writing rules

- Write for someone with no context. Define project-specific terms once in the "Glossary".
- Date every log entry (`YYYY-MM-DD`). Newest first within each section.
- Numbers need context: metric name, split, and comparison (`val F1 0.71 vs baseline 0.64`).
- Link evidence: commit hash, branch, config file, W&B/MLflow run, notebook, checkpoint report.
- Be honest. Record negative results fully; they are the most valuable part for an outsider.
- Keep "Current status" and "Next up" true right now. Rewrite them rather than appending.
- Say who decided: `(user)`, `(council, proposed)`, `(Claude, proposed)`.

## Tried and rejected: required fields

Every entry must answer:

- **What** was tried (enough detail to reproduce, link to branch/config).
- **Result** with numbers where possible.
- **Why rejected**: the actual reason (worse metric, too slow, leakage found, doesn't fit the data, too complex for the gain).
- **Revisit if**: the condition under which it might be worth trying again (more data, different metric, faster hardware). Write "no" if never.

An idea without a reason is not rejected, only abandoned. If the reason is unknown, write "reason not recorded" and ask the user.

## Keeping it readable

- Keep the file under about 400 lines. When the "Work log" grows past that, move entries older than ~2 months to `docs/progress-archive/YYYY-MM.md` and leave a one-line summary per month in their place.
- Never archive "Tried and rejected" or "Decisions log"; condense old entries to one line each instead.
- Keep "Start here" to about 10 lines.

## Reconstructing for an existing project

When setting up on a project that already has history:

1. Read README, existing notes, notebooks' markdown cells, `git log --stat` (and branch names, especially `exp/` branches), and any experiment tracker exports.
2. Draft each section from that evidence, citing commits.
3. Mark anything inferred with `(inferred)`.
4. Ask the user specifically: current best result, what they tried that didn't work and why, and what they plan next. Their answers replace the inferred lines.
