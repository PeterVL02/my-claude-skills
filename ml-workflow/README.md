# ml-workflow

A Claude Code skill for keeping AI/ML/data-science projects healthy: data pipelines, sanity checks, research, council direction reviews, a project log, a codebase knowledge graph, git hygiene, tests and CI.

## Install

Skills live in `~/.claude/skills/<name>/SKILL.md`.

```bash
# WSL / Linux / macOS
rm -rf ~/.claude/skills/ml-workflow
unzip ml-workflow.zip -d ~/.claude/skills/
```

```powershell
# Windows (native Claude Code)
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\skills\ml-workflow" -ErrorAction SilentlyContinue
Expand-Archive ml-workflow.zip -DestinationPath "$env:USERPROFILE\.claude\skills\"
```

Using both WSL and Windows? Keep one copy on the Windows side and symlink it from WSL:

```bash
ln -s /mnt/c/Users/<you>/.claude/skills/ml-workflow ~/.claude/skills/ml-workflow
```

Start a new Claude Code session afterwards; running sessions keep the old version.

### Optional companions

- **Council skill** (`llm-council`): used for direction reviews.
- **agent-skills** (addyosmani/agent-skills): TDD, debugging, code review, ADRs, etc. Used where installed.
- **graphify** (`uv tool install graphifyy`): codebase knowledge graph.
- `uv`, `ruff`, `pytest`, `pre-commit`, `gh`: assumed by the templates.

Everything degrades gracefully if a companion is missing.

## Quick start

In an ML repo:

```
/ml-workflow setup
```

This adds, without overwriting existing files:

| File | Purpose |
|---|---|
| `CLAUDE.md` | Project rules Claude reads every session: git rules, commands, compute budget, data rules |
| `PROGRESS.md` | Project log for outsiders: status, decisions, experiments, rejected ideas with reasons |
| `.claude/settings.json` | Enforced permissions: push/PR/training need approval, force-push denied |
| `.github/workflows/ci.yml` | Lint + tests on push/PR |
| `.pre-commit-config.yaml` | ruff, hygiene hooks, notebook output stripping |
| `.gitignore` additions | Data, weights, experiment logs, secrets |
| `docs/research/`, `docs/checkpoints/`, `docs/decisions/` | Research log, checkpoint reports, ADRs |

For an existing project, setup reconstructs `PROGRESS.md` from git history and asks you to fill the gaps.

Only repos where you run setup get these files; other projects are unaffected.

## Usage

| Command | What it does |
|---|---|
| `/ml-workflow` | Full **checkpoint** (below) |
| `/ml-workflow setup` | Scaffold a repo |
| `/ml-workflow sanity` | Data, leakage, model and eval sanity checks |
| `/ml-workflow pipeline` | Pipeline freshness, schemas, drift (sample run) |
| `/ml-workflow research` | One focused literature/project search, logged |
| `/ml-workflow council` | Direction, priorities, research needs, corrections |
| `/ml-workflow progress` | Update `PROGRESS.md` from recent work |
| `/ml-workflow graph` | Rebuild the graphify knowledge graph |
| `/ml-workflow ci` | Run tests/linters locally, check CI status |

Plain language also works: `/ml-workflow check the pipeline and update progress`.

### Checkpoint

A periodic health pass. On branch `chore/checkpoint-YYYY-MM-DD` it: refreshes the graph → runs tests/linters → checks the pipeline → runs sanity checks → does a research round → asks the council → runs any "Checkpoint extras" → updates `PROGRESS.md` → writes `docs/checkpoints/YYYY-MM-DD.md` → commits and stops.

The council's advice is recorded as *proposed*; nothing is acted on without your go-ahead. It's the heaviest mode (research + council take time and tokens).

## Guardrails

- **No pushing** without explicit instruction. Claude may ask to push, branch, or open a PR. Enforced by `.claude/settings.json` (`ask` rules), not just instructions.
- **Branches only**, never commits to `main`. Overrides trunk-based advice from other skills.
- **Compute budget**: no full training, sweeps, full pipeline runs or GPU jobs unless you ask. Checks use smoke configs, `--limit`, or existing logs; ~5 min per command during checkpoints. Expensive commands are listed in `CLAUDE.md` and added as `ask` rules.
- **Raw data is read-only**; data, weights and secrets are never committed.
- **Rejected ideas stay rejected**: Claude checks `PROGRESS.md` before re-proposing something.

Check that `ask` patterns match how you launch commands: `Bash(python train.py:*)` does not catch `uv run python train.py`.

## Scheduling

Run a checkpoint regularly as a scheduled task with the prompt `/ml-workflow checkpoint`:

- **Desktop scheduled task**: runs on your machine with local files, graphify and your venv. Recommended.
- **Cloud scheduled task**: only sees what's pushed to GitHub; local tools and data aren't available.
- `/loop 1h /ml-workflow sanity` inside a session for short-term repetition.

Unattended runs commit to a branch and list pending pushes in the report. If the task isn't on automatic approval it will pause at the first action needing permission.

## Customising

- **Per project**: edit the repo's `CLAUDE.md` (commands, compute budget, research focus, checkpoint extras such as a diagram-refresh skill). It overrides the skill's defaults.
- **Globally**: edit files in the skill folder.

```
ml-workflow/
├── SKILL.md                  # modes, rules, checkpoint order, report format
├── references/               # loaded only for the mode being run
│   ├── pipeline.md
│   ├── sanity-checks.md
│   ├── research.md
│   ├── council.md
│   ├── progress.md
│   └── git-and-ci.md
└── templates/                # copied into repos by setup
    ├── CLAUDE.md
    ├── PROGRESS.md
    ├── settings.json
    ├── ci.yml
    ├── pre-commit-config.yaml
    └── gitignore
```

## Troubleshooting

| Problem | Fix |
|---|---|
| `/ml-workflow` not listed | Check the path is `~/.claude/skills/ml-workflow/SKILL.md` (not nested twice); start a new session; WSL and Windows have separate homes |
| Changes to the skill not showing | Start a new session |
| Repo lacks new features after updating the skill | Setup only copies templates once; ask Claude to re-run the relevant setup step |
| Checkpoint stalls unattended | Permission prompt; set the scheduled task to automatic approval, or run interactively |
| A companion skill isn't used | Check it's installed in the same Claude install; plugin skills may be namespaced (`agent-skills:…`) |
