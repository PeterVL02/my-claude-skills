# ml-workflow

A Claude Code skill for keeping AI/ML/data-science projects healthy: data pipelines, sanity checks, research, council direction reviews, a project log, a codebase knowledge graph, git hygiene, tests and CI.

## Install

Skills live in `~/.claude/skills/<name>/SKILL.md`. The skill itself is the inner `ml-workflow/ml-workflow/` folder of this repo.

Clone the repo and link the skill folder, so `git pull` updates it. If you installed from the old zip, delete that copy first. Otherwise the link fails, or it lands nested inside the old folder:

```bash
rm -rf ~/.claude/skills/ml-workflow                       # WSL / Linux / macOS
```

```powershell
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\skills\ml-workflow"   # Windows
```

```bash
# WSL / Linux / macOS
git clone https://github.com/PeterVL02/my-claude-skills.git
mkdir -p ~/.claude/skills
ln -s "$PWD/my-claude-skills/ml-workflow/ml-workflow" ~/.claude/skills/ml-workflow
```

```powershell
# Windows (native Claude Code). A junction needs no admin rights or Developer Mode.
git clone https://github.com/PeterVL02/my-claude-skills.git
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills" | Out-Null
New-Item -ItemType Junction -Path "$env:USERPROFILE\.claude\skills\ml-workflow" -Target "$PWD\my-claude-skills\ml-workflow\ml-workflow"
```

If you'd rather not link, copy the folder instead (`cp -r` / `Copy-Item -Recurse`). To update it later, delete the copy and copy the folder again.

Using both WSL and Windows? Install on the Windows side, then symlink it from WSL:

```bash
ln -s /mnt/c/Users/<you>/.claude/skills/ml-workflow ~/.claude/skills/ml-workflow
```

Check that `~/.claude/skills/ml-workflow/SKILL.md` exists, then start a new Claude Code session. Sessions that are already running keep the old version.

### Optional companions

- **Council skill** (`llm-council`): used for direction reviews.
- **agent-skills** (addyosmani/agent-skills): `/agent-skills:test`, `review`, `plan`, `code-simplify`, `build`, `constraints`, `ship`, debugging, ADRs, etc. Used where installed. Deep checkpoints rely on `review`, `code-simplify` and `plan`.
- **ponytail**: loaded whenever Claude writes code (not for research or writing). `ponytail-audit` and `ponytail-debt` run in deep checkpoints.
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
| `CLAUDE.md` | Project rules Claude reads every session: git rules, commands, compute budget, data rules, checkpoint config |
| `PROGRESS.md` | Project log for outsiders: status, decisions, experiments, rejected ideas with reasons |
| `.claude/settings.json` | Enforced permissions: push/PR/training need approval, force-push denied |
| `.github/workflows/ci.yml` | Lint + tests on push/PR |
| `.pre-commit-config.yaml` | ruff, hygiene hooks, notebook output stripping |
| `.gitignore` additions | Data, weights, experiment logs, secrets |
| `docs/research/`, `docs/checkpoints/`, `docs/decisions/` | Research log, checkpoint reports, ADRs |

For an existing project, setup reconstructs `PROGRESS.md` from git history and asks you to fill the gaps. Setup also asks how this project's checkpoints should run (see below), and offers `/agent-skills:constraints` to set a quality bar.

Only repos where you run setup get these files; other projects are unaffected.

## Usage

| Command | What it does |
|---|---|
| `/ml-workflow` | **Checkpoint** (below); asks for the checkpoint config first if the project has none |
| `/ml-workflow setup` | Scaffold a repo |
| `/ml-workflow configure` | Change when, how often and how checkpoints run |
| `/ml-workflow deep-checkpoint` | Force a deep checkpoint now |
| `/ml-workflow sanity` | Data, leakage, model and eval sanity checks |
| `/ml-workflow pipeline` | Pipeline freshness, schemas, drift (sample run) |
| `/ml-workflow research` | One focused literature/project search, logged |
| `/ml-workflow council` | Direction, priorities, research needs, corrections |
| `/ml-workflow progress` | Update `PROGRESS.md` from recent work |
| `/ml-workflow graph` | Rebuild the graphify knowledge graph |
| `/ml-workflow ci` | Run tests/linters locally, check CI status |

Plain language also works: `/ml-workflow check the pipeline and update progress`.

### Checkpoints

There are two kinds of checkpoint:

- **Standard**: a frequent, cheap health pass. By default it refreshes the graph, then runs tests/linters, then the pipeline check, then sanity checks.
- **Deep**: less frequent. It adds a research round, the council, code review (`/agent-skills:review`), simplification (`/agent-skills:code-simplify`, `ponytail-audit`, `ponytail-debt`) and a proposed task plan (`/agent-skills:plan` → `tasks/plan.md`, ready for `/agent-skills:build` once you approve it).

Each project configures its own checkpoints in `CLAUDE.md` → "Checkpoints":
- how often each kind runs (deep: every N days, every Nth checkpoint, or manual)
- which steps each kind runs
- whether a deep checkpoint may apply safe simplifications and small review fixes (`apply-safe`: typos, unused imports and similar, each as its own commit, never touching data, metrics or configs) or only report them (`report-only`, the default)
- any context such as deadlines

Setup asks for this, and so does a plain `/ml-workflow` in a project that has no config yet. Change it later with `/ml-workflow configure`.

Both kinds work on a branch (`chore/checkpoint-YYYY-MM-DD` or `chore/deep-checkpoint-YYYY-MM-DD`), update `PROGRESS.md`, write a report to `docs/checkpoints/`, commit and stop. Advice from the council, the review and the plan is recorded as *proposed*; nothing is acted on without your go-ahead.

Checkpoints find earlier runs through their branch names too, so you don't have to merge every checkpoint branch.

**Blocking:** Critical findings from a deep checkpoint's code review are listed as checkboxes under "Blocking findings" in its report. Until each one is resolved, standard checkpoints don't run. A scheduled run just prints a notice; an interactive one asks whether to run anyway. An item counts as resolved when any of these is true:
- the fix is merged into the default branch
- it's ticked in the report
- you tell Claude it's accepted or deferred

A new deep checkpoint re-checks the open items.

## Guardrails

- **No pushing** without explicit instruction. Claude may ask to push, branch, or open a PR. Enforced by `.claude/settings.json` (`ask` rules), not just instructions.
- **Branches only**, never commits to `main`. Overrides trunk-based advice from other skills.
- **Compute budget**: no full training, sweeps, full pipeline runs or GPU jobs unless you ask. Checks use smoke configs, `--limit`, or existing logs; ~5 min per command during checkpoints. Expensive commands are listed in `CLAUDE.md` and added as `ask` rules.
- **Raw data is read-only**; data, weights and secrets are never committed.
- **Rejected ideas stay rejected**: Claude checks `PROGRESS.md` before re-proposing something.

Check that `ask` patterns match how you launch commands: `Bash(python train.py:*)` does not catch `uv run python train.py`.

## Scheduling

Run a checkpoint regularly as a scheduled task with the prompt `/ml-workflow checkpoint`. One task is enough: a standard run upgrades itself to deep when the project's config says one is due. The schedule itself lives in the task, so match it to the cadence in the project's `CLAUDE.md`.

- **Desktop scheduled task**: runs on your machine with local files, graphify and your venv. Recommended.
- **Cloud scheduled task**: only sees what's pushed to GitHub; local tools and data aren't available.
- `/loop 1h /ml-workflow sanity` inside a session for short-term repetition.

Unattended runs commit to a branch and list pending pushes in the report. If the task isn't on automatic approval it will pause at the first action needing permission.

## Customising

- **Per project**: edit the repo's `CLAUDE.md` (commands, compute budget, research focus, checkpoint config and extras such as a diagram-refresh skill). It overrides the skill's defaults.
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
│   ├── checkpoints.md        # checkpoint config interview + defaults
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
| Scheduled checkpoint does nothing and prints "blocked" | Open Critical findings from the last deep checkpoint; fix and merge, tick them in its report, or tell Claude they're accepted |
| Checkpoint stalls unattended | Permission prompt; set the scheduled task to automatic approval, or run interactively |
| A companion skill isn't used | Check it's installed in the same Claude install; plugin skills may be namespaced (`agent-skills:…`) |
