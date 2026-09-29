# Legal and ethics

This covers three things:

- the **license register**: what we use, and on what terms
- the **publish check**: what may leave the machine
- the **deep-checkpoint review**: whether the project as a whole is lawful and fair to the people in its data

This is not legal advice. The goal is to spot problems early and send real questions to a person who can answer them: a supervisor, the data owner, or the DPO / legal office. When you're unsure, flag it; don't decide it yourself.

The per-project settings live in `CLAUDE.md` → "Legal and ethics" (template: `templates/CLAUDE.md`).

## License register

The table lives in `docs/legal/licenses.md` (template: `templates/licenses.md`). One row per thing the project uses or ships:

- **models and weights**: pretrained checkpoints, fine-tuning bases, tokenizers, vocoders, embedding models
- **datasets**: training, evaluation, test sets and lexicons, including anything scraped
- **frameworks and libraries**: direct dependencies from `pyproject.toml` / `requirements*.txt`. Transitive dependencies don't need rows; check them in bulk (see the review below).
- **code**: vendored or copied code, snippets from GitHub, papers or Stack Overflow (CC BY-SA), and notebooks from elsewhere
- **services**: hosted APIs and their terms, e.g. LLM APIs whose terms restrict training on their outputs

**When to add a row:** before first use, in the same commit that adds the dependency, download or snippet. Adding a row is a docs change and is always allowed, including in `report-only` checkpoints.

**Finding the license:** read it from the source: the `LICENSE` file, the model or dataset card, the package metadata, or the terms page. Record the version or commit you checked, because licenses change between releases.
- Model licenses often restrict use (RAIL/OpenRAIL, Llama-style community licenses, "research only"). Record the use restrictions, not only the license name.
- Datasets can carry two layers of terms: the dataset license and the rights of the underlying content (e.g. audiobook recordings, news text). Note both.
- No license found means all rights reserved. Set Status to `unclear`, not `ok`.

**Status values:**
- `ok`: the license is verified and compatible with the intended use in `CLAUDE.md`
- `unclear`: not verified yet, ambiguous, or it depends on a question someone still has to answer
- `conflict`: incompatible with the intended use, e.g. NC data in a commercial project, ND content we modify, share-alike code in a closed release, or a research-only model we plan to ship

A `conflict` is a Critical finding. An `unclear` on anything we plan to publish is an Important finding.

## Publish check (sensitive projects)

This applies when `CLAUDE.md` → "Legal and ethics" says the project handles sensitive data, such as personal data, data under an agreement, confidential or unreleased data, or anything the owner restricts.

**Before every commit**, including checkpoint and fix commits:

1. Run `git diff --cached --name-only`, then read the staged diff itself. File names alone are not enough.
2. Unstage anything the "Never publish" list in `CLAUDE.md` covers. Also unstage anything that could reveal a person or the restricted data, even if the list doesn't name it:
   - data files, samples, or excerpts: audio, images, text snippets, rows, including ones pasted into tests, docs, notebooks, log output or error messages
   - identifiers: names, e-mails, national IDs, speaker/patient/user IDs, file names or paths that encode them
   - outputs derived closely from the data: generated samples, per-person metrics, confusion examples, embeddings, and weights trained on the data (unless publishing them is cleared in `CLAUDE.md`)
   - data-owner documents: agreements, consent forms, internal emails
3. **When unsure, don't commit it.** Interactive: show the user the file and the specific concern, then ask. Unattended: leave it unstaged, commit the rest, and list it under "Legal & ethics → Held back" in the report.
4. Test fixtures must be synthetic. A fixture copied or lightly edited from real data counts as data.
5. **Get approval.** Show the user the staged file list, anything held back, and any concerns, then commit only after they say yes. The exception is a standing rule under "Commits" in `CLAUDE.md`, or an explicit instruction in this session, that allows committing without asking. Unattended, with no such rule: don't commit. Leave the work staged and list it in the report.

**Before asking to push**, run the same check over everything the push would publish: `git log -p <remote>/<branch>..HEAD`, or the whole branch if it's new. Check the history too, not just the final state. A file that was deleted in a later commit is still published. If something sensitive is already in unpushed history, say so and propose a fix. Don't rewrite history without the user's go-ahead, and never rewrite pushed history.

While a legal Blocking finding from a deep checkpoint is open, don't ask to push anything it covers.

### Mechanical backstop

Claude's check is the main one. For people committing by hand, and for `git add -f` past `.gitignore`, add these local hooks to `.pre-commit-config.yaml` during setup. Fill both regexes from the project's "Never publish" list. Leave out the pygrep hook if the data has no recognisable identifier format.

```yaml
  - repo: local
    hooks:
      - id: no-sensitive-paths
        name: Block sensitive data paths (CLAUDE.md -> Legal and ethics)
        language: fail
        entry: "Sensitive path staged. Unstage it, or update the Never publish list in CLAUDE.md"
        files: '^(data|models|outputs|samples)/|\.(wav|flac|mp3|ogg|m4a|png|jpg|csv|tsv|jsonl|parquet|npy|npz|pt|pth|ckpt|safetensors|onnx)$'
        exclude: '^tests/fixtures/'   # synthetic only; see the publish check
      - id: no-identifiers
        name: Block identifiers from the sensitive dataset
        language: pygrep
        entry: '<identifier regex, e.g. speaker IDs "spk_\d{4}" or Danish CPR "\b\d{6}-?\d{4}\b">'
        types: [text]
```

Tune the path regex with the user. If the repo legitimately commits small CSVs or figures, narrow the pattern instead of dropping the hook.

## Deep-checkpoint review (legal step)

Scope: the whole project as it stands, plus anything added since the last deep checkpoint. Use only cheap operations: reading, grepping, and `git ls-files`. Checks that need compute (e.g. running a speaker-verification model over samples) go to "Suggested next steps" with a smoke-scale variant, unless `CLAUDE.md` lists one as cheap.

1. **Register complete?** Compare `docs/legal/licenses.md` against:
   - dependency files
   - model and dataset references in code and configs: `from_pretrained`, `load_dataset`, `hf_hub_download`, `torch.hub`, download URLs, `wget`/`curl` in scripts
   - vendored folders and copied code
   Add missing rows (Status `unclear` until verified). If `pip-licenses` (or `uv pip` metadata) is available, scan transitive dependencies for copyleft (GPL/AGPL) or unknown licenses, and report them only if the project ships code or binaries.
2. **Compatible with intended use?** Check every `unclear` and `conflict` row against the intended use in `CLAUDE.md` (research, publication, commercial, open release of weights). Watch for:
   - obligations the project must meet: attribution, license notices, share-alike on released code or weights
   - use restrictions in model licenses
   - terms that forbid training on API outputs
3. **Personal data (GDPR and similar)**. For each dataset with data about people:
   - Is there a documented legal basis or consent, and does it cover this use? Consent for "research on X" may not cover releasing a model.
   - Is it special-category data? Voice or face data used to identify someone is biometric data, and health data is always special-category.
   - Are storage location, access, and retention/deletion in line with the agreement? Is the project collecting more than it needs?
   - Is it pseudonymised (still personal data) or truly anonymised? Don't accept "anonymised" without evidence.
4. **Can outputs identify people?** For everything published or planned to be (weights, samples, demos, figures, papers, the repo itself), ask whether someone could be identified or re-identified, or whether the model can reproduce training data. Domain examples:
   - **Speech synthesis / voice**: released samples and voices must not be recognisable as a training speaker. Check with a speaker-verification model against the training speakers, a listening check, or target voices built so they don't match any real speaker. No released audio or text from copyrighted recordings, and no speaker metadata. Consider watermarking and a use policy against voice cloning.
   - **Images / video**: faces, plates, locations, EXIF metadata.
   - **Text / LLMs**: memorised PII or copyrighted passages in generations, and names in examples.
   - **Tabular / health**: small groups in reported tables (cells under ~10), rare combinations, and per-individual plots.
5. **Repo and history:** run `git ls-files` and look for data-like files. Run a secrets/PII scan if a tool is installed (e.g. `gitleaks detect`). Check that the backstop hooks exist and match the current "Never publish" list.
6. **Ethics beyond the law:**
   - Performance gaps across groups: speakers, accents, sex, age, dialect. Report what is measured, and what isn't but should be.
   - Foreseeable misuse of what is released, and whether a model/data card documents intended use, limitations and risks.
   - Whether the people in the data would reasonably expect this use.

**Severity:**
- **Critical** → "Blocking findings":
  - a license `conflict` on something we use or publish
  - sensitive data or identifiers in tracked files or unpushed history
  - a published or planned output that can identify people
  - personal data used without a basis that covers the use
- **Important:**
  - `unclear` licenses on published components
  - a missing model/data card for a planned release
  - pseudonymisation mistaken for anonymisation
  - an unmeasured group gap
- **Suggestion:** everything else.

Never auto-fix legal findings, even with `apply-safe`. The only changes this step makes are register rows. Questions only a person can answer go into the report as "Needs a human answer: <question> → <who>". They also go into `PROGRESS.md` → "Open questions" if that section exists.
