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
- **`Vale.Terms` + the vocabulary** (`accept.txt`) — enforces the canonical casing
  of always-capitalized proper nouns (`flagger` → `Flagger`, `externalartifact` →
  `ExternalArtifact`, `Konfiguration`, `Tanka`, …). The list is deliberately scoped
  to terms that never appear in another casing, so it adds no false positives.

Vale skips code spans, code blocks, and URLs, so backticked identifiers and links
are never flagged.

## Use it in a repo

This is a Vale **complete package** — it bundles the rules, the vocabulary, *and*
the config. `vale sync` merges all of it, so a consuming `.vale.ini` is two lines;
everything else (`BasedOnStyles`, `Vocab`, alert level) is inherited:

```ini
StylesPath = .vale/styles
Packages = https://github.com/metio/vale-config/releases/latest/download/Metio.zip
```

`.vale/styles/` is populated by `vale sync` and should be git-ignored. In CI, run
the [Vale GitHub Action](https://github.com/vale-cli/vale-action) against the docs:

```yaml
- uses: vale-cli/vale-action@v3 # pin to a SHA
  with:
    files: docs/content
```

## Layout and releases

`styles/` is the package's `StylesPath` (the `Metio/` rules and
`config/vocabularies/Metio/`); the repo-root `.vale.ini` is both the bundled
package config and the self-test config. `.github/workflows/release.yml` assembles
the [complete-package](https://docs.vale.sh/keys/packages#complete) layout — a
top-level `Metio/` directory holding `.vale.ini` + `styles/` — zips it to
`Metio.zip`, and attaches it to a calendar-versioned GitHub release;
`releases/latest/download/Metio.zip` always serves the newest.

Lint this repo's own docs against the local styles, no download needed:

```shell
vale --config=.vale.ini README.md
```
