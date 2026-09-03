---
title: Documentation
summary: Index of this project's documentation.
updated: 2026-08-05
status: living
---

# Documentation

## Important

Documents a human will want to read.

- [Getting started](guide/getting-started.md) — prepare a fresh clone.
- [Development lifecycle](guide/development-lifecycle.md) — how a change moves from bead to merge.
- [Conventions](guide/conventions.md) — how docs are written and maintained.
- [Terminology](guide/terminology.md) — the binding domain language for all code and docs.

## Others

All documents, by folder. This list is generated — run `scripts/gen-doc-index` after adding or
renaming a doc.

<!-- BEGIN generated: docs index (scripts/gen-doc-index) -->

### adapters/

- [Forge adapter](adapters/forge.md) — The code-forge host, repo, token location, and host quirks; the executable half is scripts/forge.
- [Stack adapter](adapters/stack.md) — Every command an agent runs against this project's code — build, test, lint, the CI-mirror quality gate, running the app, and the UI render check.

### adr/

- [ADR-0001 Stay on Astro 5.x for the static no-input landing page](adr/2026-09-03-stay-on-astro-5-for-static-no-input-landing-page.md) — Why the landing page stays on astro@^5.18.0 despite eight high-severity npm-audit advisories — none is reachable on a static site with no user-controlled input, and the Astro 7 upgrade currently breaks the build via an unrelated Rolldown tsconfig-resolver regression.
- [ADR index & conventions](adr/README.md) — Index of architecture decision records and the one-file-per-decision / supersede-not-edit conventions.

### guide/

- [Beads usage](guide/beads-usage.md) — How work is tracked in beads — remote Dolt state, statuses, review labels, and common commands.
- [Coding principles](guide/coding-principles.md) — Stack-independent design principles — boundaries as injection seams, measurability, determinism, typed provenance, errors per boundary, and binding terminology.
- [Conventions](guide/conventions.md) — How this project's docs are written and maintained — the frontmatter schema, the concise-content rule, and the amend-vs-supersede rubric.
- [Development lifecycle](guide/development-lifecycle.md) — How a change moves from bead to merged PR through the orchestrator, dev-implementer, and qa-verifier roles.
- [Getting started](guide/getting-started.md) — Prepare a fresh clone of basset-landing — repository access, beads tooling, the GitHub token, and the Astro toolchain.
- [Terminology](guide/terminology.md) — TEMPLATE: this project's binding domain language for code, docs, and ADRs.

<!-- END generated -->
