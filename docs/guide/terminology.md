---
title: Terminology
summary: "TEMPLATE: this project's binding domain language for code, docs, and ADRs."
updated: 2026-08-05
status: draft
---

# Terminology

<!-- TEMPLATE — replace the placeholder sections with your project's domain language.
     Keep the two structural elements: term definitions grouped by area, and the
     banned-terms table at the end. The mechanism around this doc is core kit behavior:
     CLAUDE.md and both subagents treat these terms as binding, planning words beads in
     them, and QA bounces PRs that use a banned term. -->

This project's domain language. These terms are **binding**: use them exactly — in code
(variable, type, function, module, and file names), documentation, ADRs, commit messages, bead
titles, and PR text. General industry terms (UI, API, HTTP…) are not defined here.

If a needed concept has no term yet, add it here in the same change rather than inventing an
ad-hoc name.

## <Area 1>

- **<Term>** — <definition. One concept, one name. If the term deliberately replaces a common
  industry word, say which word it replaces and why — that word is a candidate for the banned
  table below.>

## <Area 2>

- **<Term>** — <definition.>

## Deprecated — do not use

<!-- Every entry here is enforced: QA treats a banned term in new code or docs as a bounce.
     Example rows from the project this kit was extracted from:

| Banned term | Use instead |
|-------------|-------------|
| index / indexing (for building from source) | codebase ingest |
| footgun | pitfall |
-->

| Banned term | Use instead |
|-------------|-------------|
| <banned> | <replacement> |
