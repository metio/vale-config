<!--
SPDX-FileCopyrightText: The metio vale-config Authors
SPDX-License-Identifier: 0BSD
-->

# vale-config

Shared [Vale](https://vale.sh/) style for the metio documentation sites — the
naming and branding rules every repo's docs should follow, in one place. This is
the prose-linting counterpart to
[metio/renovate-config](https://github.com/metio/renovate-config): repos reference
it instead of each copying the rules.

## What it enforces

The `Metio` style (`styles/Metio/`) plus a shared vocabulary
(`styles/config/vocabularies/Metio/`):

- **`Metio.Naming`** — canonical spellings for single-token terms (`k8s` →
  `Kubernetes`, `Jaas`/`JAAS` → `JaaS`, `Stageset` → `StageSet`, …).
- **`Metio.DeprecatedNames`** — outdated repository names (`flux-stageset-controller`
  → `stageset-controller`) and `Flux CD` → `Flux`.
- **`Metio.Branding`** — rejects collective branding of the projects (`the metio
  stack`, `metio tools`, `the metio way`, …); they are independent projects.
- **`Metio.Inclusive`** — gender-neutral and inclusive language (`he/she` → `they`,
  `chairman` → `chair`, `whitelist` → `allowlist`, …) at **warning** level
  (advisory, not a hard gate).
- **`accept.txt` / `reject.txt`** — the shared vocabulary: accepted spellings and
  rejected terms, available to `Vale.Spelling` / `Vale.Terms` / `Vale.Avoid` when a
  repo opts into them.

Vale skips code spans, code blocks, and URLs, so backticked identifiers and links
are never flagged.

## Use it in a repo

Add a `.vale.ini` that pulls the published package:

```ini
StylesPath = .vale/styles
MinAlertLevel = error
Packages = https://github.com/metio/vale-config/releases/latest/download/Metio.zip
Vocab = Metio

[*.md]
BasedOnStyles = Metio
```

`.vale/styles/` is populated by `vale sync` and should be git-ignored. In CI, run
the [Vale GitHub Action](https://github.com/vale-cli/vale-action) against the docs:

```yaml
- uses: vale-cli/vale-action@reviewdog
  with:
    files: docs/content
```

## Develop

Lint this repo's own docs against the local styles, no download needed:

```shell
vale --config=.vale.ini styles README.md
# or against another repo's docs:
vale --config=.vale.ini ../flux-stageset-controller/docs/content
```

Releases are built by `.github/workflows/release.yml`, which zips `styles/` into
`Metio.zip` and attaches it to a calendar-versioned GitHub release; the
`releases/latest/download/Metio.zip` URL above always serves the newest.
