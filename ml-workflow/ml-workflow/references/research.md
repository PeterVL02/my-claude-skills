# Research: papers, projects, and recent developments

Goal: keep the project aware of relevant prior work and new results, without drowning it in links.

## Scope the search first

1. Read the project's `CLAUDE.md` "Problem" and "Research focus" sections, and `docs/research/log.md` to see what was already found.
2. Write down 2-4 specific questions before searching, e.g. "What is current SOTA for <task> on <dataset type>?", "Are there open-source implementations of <method>?", "Has anyone addressed <specific failure we see>?".
3. Only search for what is new since the last log entry, unless the user asks for a full survey.

## Where to look

- **Papers**: arXiv (relevant categories such as cs.LG, cs.CV, cs.CL, stat.ML), Semantic Scholar, Google Scholar, OpenReview (NeurIPS/ICLR/ICML), ACL Anthology for NLP.
- **Trending / code-linked papers**: Hugging Face Papers.
- **Code and models**: GitHub, Hugging Face Hub (models and datasets).
- **Benchmarks / leaderboards** relevant to the task.
- Blog posts only from primary sources (lab or author blogs), not aggregators.

## Rules

- Every item must have a real link you actually opened. Never cite a paper from memory without finding it.
- Prefer recent (last ~12 months) unless an older paper is foundational to the problem.
- Distinguish peer-reviewed from preprint.
- For code repos, note license, last commit date, and stars/activity; flag abandoned repos.
- Be honest about fit. "Related but not applicable because …" is a useful finding.
- Summarise in your own words; don't paste abstracts.

## Log format (append to `docs/research/log.md`)

```markdown
## YYYY-MM-DD — <question searched>

- **<Title>** (<venue or "preprint">, <year>) — <link>
  - What: <one line>
  - Relevance: <why it matters for this project, or why it doesn't>
  - Action: <try it / read fully / ignore / watch>
```

Keep 1-5 items per round. If a finding suggests a concrete change (new baseline, different loss, dataset fix), add it to the checkpoint's "Suggested next steps". Do not implement research ideas without the user's go-ahead.
