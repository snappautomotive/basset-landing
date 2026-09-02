# CLAUDE.md

Documentation lives in [docs/](docs/README.md).

## Workflow

See [Development lifecycle](docs/guide/development-lifecycle.md). No agent merges a pull
request — merging is the auto-merge workflow or a human.

## Domain language

This project has a defined vocabulary in [Terminology](docs/guide/terminology.md) — the
project's ubiquitous language, **binding on every agent**. Use those exact terms when naming
code (variables, types, functions, modules, files) and in documentation, ADRs, commit
messages, bead titles, and PR text. Do not coin a synonym for a term that already exists, and
never use a **banned** term from that doc. If a needed concept has no term yet, add it to the
terminology doc in the same change rather than inventing an ad-hoc name.

## Subagents

Work is executed by two subagents in [`.claude/agents/`](.claude/agents), dispatched by the
main loop (the `orchestrate` skill):

- **dev-implementer** — implements one bead on a local branch and hands off. Never pushes, opens
  a PR, merges, closes, or writes bead state.
- **qa-verifier** — adversarially verifies the work against the bead's acceptance criteria; on a
  pass it pushes the branch and opens the PR, carrying the bead's labels onto it. Never merges,
  closes, or writes bead state.

The orchestrator is the only writer of bead state and never writes product code. No agent ever
merges. See [Development lifecycle](docs/guide/development-lifecycle.md).

## Beads

All work is tracked in beads (`bd`), backed by the remote Dolt ref `refs/dolt/data` — the source
of truth. Sync with `bd dolt pull` (before working, before every claim, and again after every
push) and `bd dolt push` (after every state transition) — a push does not refresh the local read
view when history has diverged, so the following pull reconciles it. Only built-in statuses are
used; labels split into **STATUS** (lifecycle: `in-qa-review`, `in-human-review`) and
**REQUIREMENT** (plan-time, fixed: the bead permission `allows-auto-merge` — absence means human
review; `requires-adr` — overrides it, never auto-merges). QA translates the bead permission into
the PR trigger `auto-merge` (mutually exclusive with `in-human-review` on the PR) and maps
`requires-adr` to the PR signal `contains-adr`. One writer per pool. See
[Beads usage](docs/guide/beads-usage.md).

## Toolchains & commands

Every command an agent runs against this project's code — build, test, lint, the CI-mirror
quality gate, running the app, the UI render check — is defined in the **stack adapter**:
[`docs/adapters/stack.md`](docs/adapters/stack.md). Do not hardcode or guess commands; read the
adapter.

Pull-request actions — open, label, comment, reply, status — go through
[`scripts/forge`](scripts/forge) (`scripts/forge help`). The forge host, repo, token location,
and host quirks are defined in the **forge adapter**:
[`docs/adapters/forge.md`](docs/adapters/forge.md). See
[Getting started](docs/guide/getting-started.md).
