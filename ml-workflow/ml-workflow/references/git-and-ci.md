# Git structure, tests, and CI/CD

## Standard repo layout

Adapt to what exists; don't move code without asking.

```
project/
├── CLAUDE.md                 # project memory for Claude (see template)
├── PROGRESS.md               # project log for outsiders (see references/progress.md)
├── README.md
├── pyproject.toml            # deps + tool config (ruff, pytest, mypy)
├── .claude/settings.json     # permissions (push/PR need approval)
├── .github/workflows/ci.yml
├── .pre-commit-config.yaml
├── configs/                  # experiment / pipeline configs (YAML)
├── data/                     # gitignored
│   ├── raw/                  # immutable
│   ├── interim/
│   └── processed/
├── docs/
│   ├── research/log.md
│   ├── checkpoints/
│   ├── progress-archive/     # old PROGRESS.md work-log entries
│   ├── decisions/            # short ADRs for big choices
│   └── legal/licenses.md     # license register (see references/legal-ethics.md)
├── notebooks/                # exploration only; logic moves to src/
├── models/                   # gitignored weights/checkpoints
├── reports/                  # figures, metrics, data stats
├── scripts/                  # thin CLI entry points
├── src/<package>/
│   ├── data/                 # loading + pipeline stages
│   ├── features/
│   ├── models/
│   ├── training/
│   └── evaluation/
└── tests/
    ├── unit/
    ├── sanity/               # data + model sanity checks
    └── conftest.py
```

## Branches and commits

- `main` is always green. No direct commits. Every experiment and checkpoint branches off main, so a broken main breaks all of them.
- Branch names: `feat/<short>`, `fix/<short>`, `exp/<short>` (experiments), `data/<short>` (pipeline changes), `docs/<short>`, `chore/<short>`. The prefixes are how checkpoints and `PROGRESS.md` reconstruction find experiments later.
- Conventional commits: `feat: …`, `fix: …`, `data: …`, `exp: …`, `test: …`, `ci: …`, `docs: …`, `chore: …`, `refactor: …`. With these, `git log` can answer "when did the data change?" at a glance.
- Small, focused commits. Don't mix a refactor with a behaviour change. When a metric moves, the cause can then be found with `git bisect` or a single revert.
- One PR per branch, with a description of what changed, how it was tested, and any metric changes. The PR is where an outsider learns why a number changed.
- Before asking to push or open a PR, run `/agent-skills:review` (and `ponytail-review` on the diff) if installed, and make sure `PROGRESS.md` reflects the branch's work. In sensitive projects, also run the publish check over everything the push would publish (`references/legal-ethics.md`).
- The `git-workflow-and-versioning` companion skill's advice on atomic commits applies; its trunk-based model does not override the branch rule above.

## Remote actions (important)

- Local branches and commits: allowed. In sensitive projects, each commit needs approval (see `references/legal-ethics.md`).
- `git push`, `gh pr create`, `gh pr merge`, tags, releases: **only when the user explicitly asks in the current session.** You may ask: "Want me to push `feat/x` and open a PR?"
- The one exception: a checkpoint pushes its own branch when auto-push applies (`references/checkpoints.md` → "Pushing checkpoint branches"). It never opens a PR on its own.
- Never force-push, never push to `main`.

## Tests

- `pytest` with `tests/unit` (fast, pure functions, transforms) and `tests/sanity` (data/model checks on tiny fixtures).
- Use small synthetic fixtures in `tests/fixtures/`; CI must not need real data or a GPU. CI runners have neither, and real data must not be committed (sensitive projects: not even excerpts).
- Mark slow or data-dependent tests with `@pytest.mark.slow` and skip them in CI by default. A CI run that takes too long stops getting waited for.
- When fixing a bug, add a test that fails before the fix. Otherwise there's no proof the fix fixes anything, and no guard against the bug coming back.
- Aim for tests on every pipeline transform and every metric function; don't chase coverage numbers on training loops. Silent ML bugs live in transforms and metrics. Training loops are better checked by the sanity checks (overfit one batch, loss decreases).

## CI procedure (ci mode)

1. Check `.github/workflows/` exists and matches the template's jobs (lint, type-check, tests). If missing, propose adding `templates/ci.yml`.
2. Run the same commands locally: `ruff check .`, `ruff format --check .`, `mypy src` (if configured), `pytest -m "not slow"`.
3. If `gh` is available and authenticated, read the latest CI runs (`gh run list --limit 5`) — this is read-only and allowed. Report failures and their cause.
4. Fix trivial failures (formatting, imports, outdated test fixtures) on a branch. Report anything that needs a judgement call.
5. Check that dependency versions are pinned (lockfile present) and flag stale or vulnerable ones if a tool (`uv pip list --outdated`, `pip-audit`) is available.
