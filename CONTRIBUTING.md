<!--
SPDX-FileCopyrightText: The metio vale-config Authors
SPDX-License-Identifier: 0BSD
-->

# Contributing

This repository holds the shared [Vale](https://vale.sh/) style for the metio
documentation sites. A change here lands in every consuming repo's next CI run, so
the bar for a new rule is: **it must add no false positives to the existing metio
docs.**

## Layout

```
.vale.ini                              the bundled package config (and self-test config)
styles/
  Metio/*.yml                          the custom rules
  config/vocabularies/Metio/accept.txt the shared vocabulary (Vale.Terms)
```

`.github/workflows/release.yml` assembles these into a Vale
[complete package](https://docs.vale.sh/keys/packages#complete) — a top-level
`Metio/` directory holding `.vale.ini` + `styles/` — and publishes `Metio.zip`.
`vale sync` merges that whole `styles/` tree into a consumer's StylesPath, so the
rules, the vocabulary, and the config all travel together.

## Add or change a rule

- **Custom rule** → a `.yml` in `styles/Metio/`. Use `substitution` for
  single-token wrong→right fixes, `existence` for multi-word or banned phrases
  (Vale's `substitution` only matches single word tokens reliably).
- **Vocabulary term** → a line in `styles/config/vocabularies/Metio/accept.txt`.
  `Vale.Terms` enforces the listed casing, so **only add terms that are *always*
  written one way.** A term that legitimately appears in another casing
  (`Kustomize` the tool vs `kustomize` the CLI, `Bucket` the kind vs a generic
  bucket, a project's own lowercase name like `grafana-operator`) will produce
  false positives — keep it out.
- **Level**: `error` for objective naming/branding rules (these gate CI); `warning`
  for stylistic guidance (marketing, hedging, inclusive language) — advisory, never
  blocking.

## Test before opening a PR

Vale skips code spans and URLs, so backticked identifiers are safe. Lint this
repo's own styles and, crucially, a real consumer's docs — a new rule must report
**zero** findings against already-clean docs:

```shell
# self-test (rules + vocabulary, no download needed)
vale --config=.vale.ini README.md

# against a consumer's docs — expect 0 findings
vale --config=.vale.ini ../flux-stageset-controller/docs/content
vale --config=.vale.ini ../jaas/docs/content
```

No Vale on the host? Run it in a container:

```shell
podman run --rm \
  -v "$PWD:/cfg:ro" -v "$PWD/../flux-stageset-controller/docs/content:/docs:ro" \
  docker.io/jdkato/vale:latest --config=/cfg/.vale.ini --no-exit /docs
```

Also confirm the rule fires on a deliberately bad sample, so you know it works.

## Releasing

`release.yml` runs on every push to `main` that touches `styles/**` (and on
`workflow_dispatch`). It builds the complete package, signs it with cosign keyless,
and publishes a GitHub release; `releases/latest/download/Metio.zip` always serves
the newest.

**Immutable releases are enabled on this repo**, so a tag can never be recreated.
The release version is therefore `<date>+<HHMMSS>` (e.g. `2026.6.16+115530`) — the
time suffix makes every tag unique, so same-day re-runs don't collide.

If the repo also has a **"Restrict creations" tag ruleset**, the GitHub Actions bot
cannot create release tags until it is allowed. Add the repository's **GitHub
Actions** as a **bypass actor** on that ruleset (Settings → Rules → the tag ruleset
→ Bypass list), or publish manually as an admin:

```shell
# build the package the way the workflow does, then:
gh release create "$(date +'%Y.%-m.%-d+%-H%M%S')" \
  Metio.zip SHA256SUMS Metio.zip.cosign.bundle --latest
```

## Licensing

The repository is [REUSE](https://reuse.software/)-compliant and `0BSD`. `.yml`,
`.ini`, and Markdown files carry an SPDX header; the vocabulary `.txt` files (which
Vale reads line-by-line and cannot carry comments) are covered by `REUSE.toml`.
