---
title: Getting started
summary: Prepare a fresh clone of basset-landing — repository access, beads tooling, the GitHub token, and the Astro toolchain.
updated: 2026-09-02
status: living
---

# Getting started

Prepare a fresh clone before working on this project. The **setup** skill walks these
checks automatically.

## Repository access

- SSH or HTTPS access to `github.com`.
- Clone the repository.
- Install the `gh` CLI (`brew install gh`) and authenticate: `gh auth login`. The
  account needs at least `repo` scope for this repository. `gh`'s keyring supplies auth
  to `scripts/forge`.

## Ticketing (beads)

beads (`bd`) tracks all work; `bv` is its TUI viewer.

Install:

```bash
brew install beads
brew install dicklesworthstone/tap/bv
```

Fetch bead state:

```bash
bd dolt pull
```

View tasks by running `bv` from the repository root; update it with `bv --update`.

## Forge configuration (pull requests)

Agents open, label, and comment on pull requests through `scripts/forge`, which wraps
the `gh` CLI.

1. Copy [`.secrets/forge.env.example`](../../.secrets/forge.env.example) to
   `.secrets/forge.env` (gitignored) and set `FORGE_REPO` to `owner/repo`.
2. Local auth is handled by `gh auth login`; a `FORGE_TOKEN` in the env is only needed
   if you want to override the keyring (rare) or in CI. Host specifics:
   [forge adapter](../adapters/forge.md).
3. Run `scripts/forge seed-labels` once to create the six workflow labels on the repo.

## Build, test, run

All commands live in the [stack adapter](../adapters/stack.md).

Install the toolchain:

```bash
# Node (managed via nvm; version pinned in .nvmrc)
nvm install
npm install
```

Then any of:

```bash
npm run dev       # dev server with hot reload
npm run build     # produce dist/ (what GitHub Pages serves)
npm run preview   # serve dist/ locally, matches Pages
npm run check     # astro check (typecheck)
```

## Generated artifacts

Some checked-in files are generated from a single source and must be regenerated when
that source changes. Each generator has a `--check` mode that CI runs to keep the
checked-in copy current.

- **Docs index** — the generated list in [`docs/README.md`](../README.md) is built from
  each doc's frontmatter. Regenerate with `scripts/gen-doc-index` after adding or
  renaming a doc.
