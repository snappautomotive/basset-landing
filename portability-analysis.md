# basset-agent-kit → drop-in kit: portability analysis

Goal: identify every project-specific element so the kit can be generalized into a
drop-in agentic-workflow starter for arbitrary projects.

## Decisions so far

- **2026-08-07 — Second dry run: GitHub forge adapter exists.** The Fotospots drop-in
  (`~/Dev/Fotospots`, branch `kit/dropin`) runs the kit in forge mode against GitHub via a
  gh-backed `scripts/forge` implementing the same verb interface. Deliberate deviations from
  the Forgejo original: `reply <pr> <comment-id> <body>` (GitHub threads by comment id, not
  review-id/path/position); `automerge <pr>` verb using GitHub native auto-merge instead of a
  label-watching workflow (the `auto-merge` label becomes a visible marker only); `upload`
  unsupported (no GitHub API for PR-body images — screenshots are committed on the bead branch
  under docs/qa-screenshots/<bead-id>/ and linked by path); no server-side deny-list backstop —
  agent-governance paths are never-auto-merge by QA policy instead. Portability bug found by
  smoke test: empty-array expansion under `set -u` breaks on macOS bash 3.2 — use
  `${ARR[@]+"${ARR[@]}"}`. This adapter is the upstream candidate for the kit's second forge
  implementation.

- **2026-08-05 — First dry run: local mode is a real kit variant.** The CleverFunding
  prototype (`cf_work_bench/portal-proto`) runs the kit with no forge and no remote: QA's
  publishing step is removed, and the orchestrator squash-merges into `main` on a QA pass —
  unattended for `allows-auto-merge` beads, after asking the human in chat otherwise
  (`requires-adr` always asks and demands the ADR in the change). No dolt sync (local bead DB,
  single writer). Model pinning per role (dev/QA `model:` frontmatter; orchestrator = session
  model). Consider upstreaming a local-mode toggle so this adaptation is not hand-made per
  project.

- **2026-08-05 — Strategy: core + adapters, built.** The generalized kit lives in
  `agent-kit/`. Core (process, roles, labels, beads discipline) is verbatim-portable;
  two adapters hold project specifics: `docs/adapters/stack.md` (all build/test/lint/CI
  commands) and `docs/adapters/forge.md` (+ `scripts/forge`, now configured entirely via
  `.secrets/forge.env` — no hardcoded host/repo). `docs/guide/terminology.md` ships as a
  content template. The previously-missing `scripts/gen-doc-index` was written fresh
  (generic, tested). Skills renamed without the basset prefix: orchestrate, planning,
  review, docs-gardening, setup (setup now also validates adapter fill-in). ci.yml is a
  skeleton whose quality-gate job fails loudly until filled from the stack adapter;
  auto-merge.yml deny-list now also protects `docs/adapters/`.

- **2026-08-05 — Tracker stays beads.** Michel continues with beads (`bd`) in other
  projects, so the tracker axis (#3) is FIXED, not parameterized: all `bd` commands,
  the pull→push→pull sync discipline, and beads-usage.md remain in the core verbatim.
  No tracker adapter. Remaining swap points: **stack** (build/test/lint commands) and
  **forge** (host wrapper); if future projects also stay on Forgejo/Codefloe, the forge
  adapter reduces to configuration (`FORGE_API`/`FORGE_REPO` + token setup).

## The seven project-bound axes

Nearly every project-specific string in the kit falls on one of these axes. Parameterize
these and the rest is mechanical:

| # | Axis | basset value | Where it's woven in |
|---|------|--------------|---------------------|
| 1 | **Project identity** | `basset`, automotive DLT log analysis | Everywhere — file names (`basset-*` skills), prose, examples |
| 2 | **Language & toolchain** | Rust/`cargo` (+fmt/clippy/nextest/deny), Node/Vite/Tauri frontend | CLAUDE.md §Toolchains, dev-implementer §3, qa-verifier §4, rust-practices.md, ci.yml |
| 3 | **Ticket tracker** — *FIXED: beads everywhere (see Decisions)* | beads (`bd`) backed by Dolt (`refs/dolt/data`), pull→change→push→pull discipline | CLAUDE.md §Beads, orchestrate/planning skills, development-lifecycle.md, beads-usage.md, setup skill |
| 4 | **Code forge** | Forgejo at `git.snappautomotive.net`, repo `snappos-dlt/basset`, `scripts/forge`, `FORGE_TOKEN` in `.secrets/forge.env` | scripts/forge, qa-verifier §Publishing, orchestrate skill, getting-started.md, all workflows |
| 5 | **CI system** | Forgejo Actions, custom baked `basset-ci` image, org PAT secrets | .forgejo/workflows/*, qa-verifier §4 (CI mirror), auto-merge deny-list |
| 6 | **Domain vocabulary** | terminology.md (ingest, causal graph, banned: index/substrate/footgun) | CLAUDE.md §Domain language, both agents, planning/review skills, coding-principles.md |
| 7 | **Verification harnesses** | `scripts/eval` + `BASSET_EVAL_DATA` (held-out eval), `run-mac.sh` (Tauri render check) | qa-verifier §5, getting-started.md, evaluation-methodology.md, development-lifecycle.md |

## What is already generic (keep nearly verbatim)

The **process architecture** is project-agnostic and is the actual value of the kit:

- **Role split & boundaries** — orchestrator (owns ticket state + git sync, never writes
  code) → dev-implementer (implements, commits locally, never pushes) → qa-verifier
  (adversarial gate, sole publisher: pushes + opens PR on pass). "No agent ever merges."
- **Label taxonomy** — STATUS (`in-qa-review`, `in-human-review`) vs REQUIREMENT
  (`allows-auto-merge`, `requires-adr`) vs PR trigger/signal (`auto-merge`,
  `contains-adr`), the translate-don't-mirror rule, mutual-exclusion invariants, the
  feedback-round label swap owned by the orchestrator.
- **Worktree isolation** — one worktree per ticket at `.claude/worktrees/<id>`, main
  clone stays on `main`, the single-HEAD collision rationale.
- **Merge hygiene** — keep PRs current by merging `main` in (never rebase — force-push
  detaches review comments), auto-merge deny-list as integrity backstop, block-on-outdated
  handling.
- **Query-state-never-assume**, evidence-over-claims, cross-cutting-changes-sweep-every-
  call-site, stop-and-escalate rules.
- **Docs conventions** — frontmatter schema, minor-vs-substantive rubric, ADR
  supersede-never-edit, generated index with `--check` in CI, docs-gardening sweep.
- **Setup skill pattern** — verify environment + seed load-bearing rules into machine
  memory, "repo is source of truth, memory is a recall trigger".
- **PR body conventions** — What / Before→After with real values / Why / Verified;
  body-via-file for shell metacharacters.

## File-by-file classification

### A. Generic — keep, light renaming only

| File | Notes |
|------|-------|
| `AGENTS.md` | Pure pointer file. |
| `docs/guide/development-lifecycle.md` | The core process spec. Only swap: `bd`/bead nouns, `bd/<id>` branch prefix, Dolt sync lines, forge nouns, the local-render-check paragraph (harness-specific), `block_on_outdated_branch` (Forgejo rule name). |
| `docs/guide/conventions.md` | Generic except `scripts/gen-doc-index` (script **not included in kit** — see Gaps) and Forgejo-renders-frontmatter note. |
| `docs/adr/README.md` §Conventions | Conventions half is generic; index half is example content. Note: references stale label name `adr` (pre-cutover) — bug to fix regardless. |
| `.claude/skills/docs-gardening/SKILL.md` | Generic sweep; swap `bd` commands, gen-doc-index reference, "Android-Developer-docs-style" phrasing. |
| `.claude/skills/basset-review/SKILL.md` | Generic except: name, forge commands, `reviews/` auto-merge allow-list claim (repo-policy-specific), terminology link. |
| `.github/copilot-instructions.md`, `.github/instructions/mermaid.instructions.md` | Vendored generic tooling instructions. Keep or drop wholesale. |

### B. Parameterizable — generic skeleton, project-specific inserts

| File | Project-specific parts |
|------|------------------------|
| `CLAUDE.md` | §Domain language (points at terminology.md — keep mechanism, template the content); §Beads (tracker-specific sync discipline); §Toolchains (`cargo` commands, `scripts/forge`). Structure itself is the template. |
| `.claude/agents/dev-implementer.md` | "for basset"; §0 reading list (paths generic, keep); §3 verify commands (`cargo build/test/clippy`); §4 commit example (`feat(decode): parse DLT storage header [bd-abc]`, `Bead:` trailer); `bd --readonly show`, `bd ready`. Branch prefix `bd/<id>`. |
| `.claude/agents/qa-verifier.md` | The most project-dense file. §4 **CI mirror**: hardcoded job list (gen-doc-index, cargo×6, npm typecheck/build, desktop feature) — though it already says "the yml is the source of truth", the worked list must be regenerated per project. §5 + §Screenshots: `run-mac.sh`, Tauri IPC, `crates/client/ui/**` path trigger, stub server. §Publishing: `scripts/forge` invocations, Forgejo no-hard-wrap rule, private-repo curl auth note. Frontmatter `tools:` includes browser MCP tools (project needs UI driving). |
| `.claude/skills/basset-orchestrate/SKILL.md` | All `bd`/Dolt commands; `scripts/forge prs/status/comments/reply/label/unlabel`; branch prefix; Forgejo comment-anchoring rationale (rebase warning — generic for Gitea/Forgejo/GitHub, keep). Loop/labels/feedback-round structure is fully generic. |
| `.claude/skills/basset-planning/SKILL.md` | `bd create/label/propagate` syntax; bead types list; the "suggest auto-merge vs human review" examples name basset subsystems ("the matching algorithm"). Alignment-before-filing process is generic. |
| `.claude/skills/setup-basset/SKILL.md` | §1 Environment: `bd`/`bv` brew installs, Dolt pull, forge token steps. §2 Rules list is generic **except** rule wording references beads/dolt. The seeding-manifest pattern is generic. |
| `docs/guide/beads-usage.md` | Tracker fixed to beads (see Decisions) → this file is CORE: keep as-is, only delete §Label migration (one-time historical cutover — dead weight in a template). |
| `docs/guide/getting-started.md` | Template with placeholder sections: repo host/SSH, tracker install, forge token, run-the-app (`scripts/dev` — **not in kit**), render harness, eval data, generated artifacts. Every concrete value is basset's. |
| `scripts/forge` | Forgejo/Gitea API wrapper. Hardcoded defaults: `FORGE_API=https://git.snappautomotive.net/api/v1`, `FORGE_REPO=snappos-dlt/basset` (env-overridable already). `seed-labels` label set + descriptions are the generic taxonomy — keep. For GitHub, entire script would be replaced by a `gh` adapter with the same subcommand interface (comments/reply/pr-create/pr-edit/upload/label/unlabel/prs/status/seed-labels) — the **interface** is the portable artifact. |
| `.forgejo/workflows/auto-merge.yml` | The pattern (label trigger + `contains-adr` exclusion + deny-list backstop + merge-when-checks-succeed) is generic. Project-specific: deny-list entries `^db/`, `deny.toml`; Forgejo API shapes; would need a GitHub Actions twin. Deny entries for `.forgejo/`, `.claude/agents|skills/`, `CLAUDE|AGENTS.md` are generic self-protection — keep. |
| `.claude/settings.local.json` | Machine-local; allowlist names `bd dolt`, `scripts/forge`, plus a stray `gh issue *` (GitHub — likely leftover). Ship as an example/template. |

### C. Project-specific — replace with per-project content (keep as worked examples)

| File(s) | Role in template |
|---------|------------------|
| `docs/guide/terminology.md` | Replace content; keep the **format**: binding-terms sections + "Deprecated — do not use" banned-term table. This doc is load-bearing for CLAUDE.md/agents/planning. |
| `docs/guide/architecture.md` | Wholly basset. Placeholder: "describe your system here". |
| `docs/guide/rust-practices.md` | Language adapter. A template would have `<lang>-practices.md` per stack (the quality-gate/error-handling/testing/deps section headings generalize well). |
| `docs/guide/coding-principles.md` | ~60% generic principles (measurability drives boundaries, determinism, typed provenance, small verifiable units, terminology binding); ~40% basset specifics (crate split, ADR-0002, model/api/server/client). Worth splitting: generic principles → template, examples → replace. |
| `docs/guide/evaluation-methodology.md` | Basset's eval harness. The **anti-cheat standing rule** (held-out data, agents call the wrapper, metrics-only output, baseline-regression guard, never in CI) is a generic pattern worth extracting to a short template doc. |
| `docs/adr/*` (16 ADRs), `docs/design/*` (9 docs) | Reference content only — the manifest says they're included as worked examples of format/rigor. In a drop-in: keep 1 exemplar ADR (or a skeleton), drop the rest. |
| `docs/README.md` | Regenerate per project (generated file). |
| `scripts/eval` | Basset-specific (`BASSET_EVAL_DATA`, cargo run). Keep as example of the metrics-only wrapper pattern, or drop. |
| `.forgejo/workflows/ci.yml`, `ci-image.yml` | Deeply basset/runner-specific (baked image, act_runner bug workarounds, store-integration path gate, boot smoke). In a template: replace with a minimal CI skeleton + the auto-merge-required-check contract. The QA "mirror CI locally" contract only requires *some* ci.yml to exist as source of truth. |

## Cross-cutting strings to parameterize

- **Names**: `basset` (project), `basset-*` skill names, `bd-abc` example ids.
- **Hosts/repos**: `git.snappautomotive.net`, `snappos-dlt/basset`, `snappos-dlt/basset-eval.git`.
- **Branch/commit conventions**: branch prefix `bd/<id>`, `reviews/<slug>`; `[<id>]` subject tag + `Bead: <id>` trailer (rename trailer per tracker, e.g. `Refs:`/`Closes:`).
- **Paths**: `.claude/worktrees/<id>` (generic, keep); `.secrets/forge.env`; `crates/**`, `db/migrations/**`, `contract/*` (basset).
- **Tracker verbs**: `bd dolt pull/push`, `bd ready`, `bd update --claim`, `bd label add/remove/propagate`, `bd close --reason`, `bd --readonly show`, `bd human`, `bd defer`.
- **Forge verbs**: `scripts/forge <cmd>` set — treat the subcommand list as the portable adapter interface.
- **Label names**: generic set — keep verbatim (`in-qa-review`, `in-human-review`, `allows-auto-merge`, `auto-merge`, `requires-adr`, `contains-adr`).
- **Build/test/lint commands**: cargo/npm set → per-stack insert.

## Gaps: files referenced but not in the kit

A drop-in consumer will hit dangling references. Either include templates or stub the references:

- `scripts/gen-doc-index` — referenced by conventions.md, docs-gardening, ci.yml, qa-verifier §4. **Generic and load-bearing; should be included.**
- `scripts/dev` — getting-started §Run the app.
- `crates/client/e2e/run-mac.sh` + client-qa-verification design doc's harness — qa-verifier §5/§Screenshots.
- `.forgejo/ci-image/**` — ci.yml/ci-image.yml.
- `reviews/README.md` — basset-review skill (it self-heals: "create if missing").
- `.secrets/forge.env` — deliberately excluded; template needs a `.secrets/forge.env.example`.
- `.gitignore` entries assumed: `.claude/worktrees/`, `.beads/`, `.secrets/`.

## Incidental fixes noticed

- `docs/adr/README.md` §Conventions still says "carries the `adr` label" — stale vs the
  `requires-adr`/`contains-adr` cutover in beads-usage.md.
- `settings.local.json` allows `gh issue *` — GitHub leftover in a Forgejo project.
- `beads-usage.md` §Label migration is a one-time historical note — drop from any template.

## Suggested parameterization strategy (to discuss)

Three viable shapes, in increasing effort:

1. **Find-and-replace template** — keep the kit's structure; mark every project-bound
   string with `{{PLACEHOLDER}}`s and ship a `setup` skill that walks the user through
   filling them (it already has `setup-basset` to evolve into this).
2. **Core + adapters** — split each doc into a generic core and adapter surfaces:
   *forge adapter* (`scripts/forge` interface implemented for Forgejo vs `gh`) and
   *stack adapter* (build/test/lint commands + `<lang>-practices.md`). The tracker is
   fixed to beads and stays in the core (see Decisions). The label taxonomy, lifecycle,
   and role boundaries stay shared.
3. **Generator** — a single `project.yaml` (name, forge, tracker, stack, commands,
   terminology seed) from which the kit files are rendered.

Option 2 preserves the most value with the least ongoing sync burden; option 1 is the
fastest to ship.
