# Council reviews

Goal: regularly step back and ask whether the project is heading the right way, before effort piles up in the wrong direction. The council skill (`llm-council`) runs a question past several independent advisors, has them review each other, and gives a verdict.

## When to run

- Checkpoints whose config includes the council step (by default, deep checkpoints only; see `references/checkpoints.md`).
- At real decision points: choosing between approaches, after a disappointing or surprisingly good result, before a large refactor or a new data source, when progress has stalled.
- When the user asks ("council this", "are we on the right track?").

Not for small or factual questions. A council run is expensive; one well-framed question beats several vague ones.

## Prepare the brief

The advisors start with no context, so the brief must stand on its own. Keep it to about one page:

1. **Goal**: problem, primary metric, current best vs baseline (from `PROGRESS.md` → "Current status").
2. **What changed recently**: short bullets from git log and the checkpoint findings so far.
3. **Evidence**: test/CI state, pipeline and sanity results, especially any warnings or fails.
4. **Research**: new findings from this round and relevant leads.
5. **Already rejected**: the "Tried and rejected" list with one-line reasons, so the council doesn't re-suggest them.
6. **Open questions and current plan**: what the team intends to do next.
7. **The questions** (below).

Only include facts you have verified. Mark uncertain things as uncertain.

## Standard checkpoint questions

1. **Direction**: Given the goal and evidence, is the current approach the right one? Answer: on track / adjust / pivot, with reasons.
2. **Prioritisation**: Of the planned next steps and open leads, which 1-3 matter most right now, and what should be dropped or deferred?
3. **Research needed?**: Is there a gap where we are guessing and should read the literature or look for existing implementations first? Name the specific question to research.
4. **Corrections**: Anything in the evidence that looks wrong or risky (possible leakage, weak baseline, overfitting to val, metric mismatch, wasted effort)?
5. **Kill criteria**: For the current main approach, what result would tell us to abandon it?

For a decision-point review, replace these with the specific decision and the options.

## Recording the outcome

- Checkpoint report → "Council verdict": direction, top priorities, research needed, key corrections.
- `PROGRESS.md`:
  - "Decisions log": the verdict and the reason, dated, marked `(council)`. Record the user's final decision separately once they make it; the council's view is advice, not a decision.
  - "Next up": the council's priorities, marked as proposed until the user accepts them.
  - "Tried and rejected": anything the council recommends dropping stays in "Ideas and leads" marked `council suggests dropping` until the user agrees.
  - "Open questions": research questions it raised.
- If the council disagrees with the user's stated plan, say so plainly in the report. Don't soften it.
- Do not act on the verdict during the checkpoint; it only shapes suggested next steps.

If the council skill isn't installed, do a shorter version yourself: answer the five questions honestly, and label the section "Self-review (no council)".
