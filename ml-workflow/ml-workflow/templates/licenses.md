# License register

Everything this project uses or ships, and the terms it comes under. Add a row **before** first use, in the same commit. Maintained with the `ml-workflow` skill (`references/legal-ethics.md`).

Intended use: see `CLAUDE.md` → "Legal and ethics".

Status: `ok` (verified, compatible with intended use) · `unclear` (not verified or open question) · `conflict` (incompatible, must be resolved)

| Component | Type | Version / source | License | Obligations and restrictions | How we use it | Status | Checked |
|---|---|---|---|---|---|---|---|
| <e.g. LibriTTS> | dataset | <v1.0, openslr.org/60> | CC BY 4.0 | Attribution; underlying LibriVox audio is public domain | Training | ok | YYYY-MM-DD |
| <e.g. some-org/some-model> | model | <HF, commit abc123> | <OpenRAIL-M> | <Use restrictions (no impersonation, …); pass them on to downstream users> | Fine-tuning base | unclear | YYYY-MM-DD |
| <e.g. torch> | framework | <2.4> | BSD-3-Clause | Keep license notice if redistributed | Training | ok | YYYY-MM-DD |
| <e.g. src/pkg/utils/stft.py> | code | <copied from github.com/x/y@sha> | <MIT> | Keep copyright header | Feature extraction | ok | YYYY-MM-DD |

Types: model, dataset, framework, library, code, service.
