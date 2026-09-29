# Setup

Setup runs once per repo, and again when the user asks to re-run a step after the skill has changed. It always runs interactively, because most steps need answers from the user.

1. Inspect the repo: language, package manager (uv/poetry/pip/conda), existing layout, existing CI, existing `CLAUDE.md`.
2. Copy templates from this skill's `templates/` folder **without overwriting**. If a target file exists, merge in the missing parts and show the user the diff.
   - `templates/CLAUDE.md` → `./CLAUDE.md`. Fill in the placeholders from what you found; ask about anything you couldn't infer.
   - `templates/PROGRESS.md` → `./PROGRESS.md` (see step 7)
   - `templates/settings.json` → `./.claude/settings.json`
   - `templates/ci.yml` → `./.github/workflows/ci.yml`
   - `templates/pre-commit-config.yaml` → `./.pre-commit-config.yaml`
   - `templates/gitignore` → append the missing lines to `./.gitignore`
3. **Compute budget.** Find the expensive entry points: training, sweeps, full pipeline runs, evaluation on the full test set.
   - List them in `CLAUDE.md` → "Compute budget", each with its cheap smoke variant.
   - Add each one to the `ask` list in `.claude/settings.json` (e.g. `"Bash(python -m mypkg.train *)"`), so it always needs approval.
   - Show the user the list and ask whether anything is missing.
4. **Legal and ethics.** Run the interview in `references/legal-ethics.md` → "Configure". It fills `CLAUDE.md` → "Legal and ethics", creates `docs/legal/licenses.md`, and adds the extra safeguards if the project has sensitive data.
5. **Checkpoints.** Fill `CLAUDE.md` → "Checkpoints" by running the interview in `references/checkpoints.md`. Do this after step 4, because whether checkpoint branches get pushed automatically depends on the legal answers.
   - If the repo has no `CONSTRAINTS.md`, offer `/agent-skills:constraints` to set a quality bar. Ask; don't run it unprompted.
6. **Folders.** Create any missing standard folders from `references/git-and-ci.md`, with `.gitkeep` where they'd be empty. This includes `docs/research/log.md`, `docs/checkpoints/`, `docs/decisions/` and `docs/legal/`. Don't move existing code without asking.
7. **PROGRESS.md.** For an existing project, reconstruct it from git history, the README, notebooks and existing notes (see `references/progress.md` → "Reconstructing"). Then ask the user to fill the gaps, especially rejected ideas and why they were rejected.
8. **Graph.** Set up graphify (see SKILL.md → **graph**).
9. **Green start.** Run the tests and pre-commit once, so the user starts from a green state, or report what fails.
10. **Commit.** Commit on a branch `chore/ml-workflow-setup` (with the user's approval if the project is sensitive), then ask whether to push and open a PR. Setup never pushes automatically.
