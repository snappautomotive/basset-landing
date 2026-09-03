---
title: Stack adapter
summary: Every command an agent runs against this project's code — build, test, lint, the CI-mirror quality gate, running the app, and the UI render check.
updated: 2026-09-02
status: living
---

# Stack adapter

Every command an agent runs against this project's code lives here. Core kit files
(CLAUDE.md, the subagents, the skills) reference the section names below and never
hardcode commands — so swapping the stack means editing this one file.

## Build & language

**Astro** (static-site framework) with plain CSS, no Tailwind. Language: TypeScript for
`.astro` component scripts, plain CSS in `<style>` blocks or `src/styles/*.css`. Package
manager: `npm`. Node version: pinned in `.nvmrc` (currently 22.x LTS).

Deploy target: **GitHub Pages** (static). `astro build` writes the static site to `dist/`;
the `deploy` workflow uploads that as a Pages artifact.

Fonts: **IBM Plex Sans** and **IBM Plex Mono** via Google Fonts.

## Dev self-verify

The cheapest real checks a dev-implementer runs before handing off. One command per
line, cheapest first.

- typecheck: `npm run check` (wraps `astro check`)
- build: `npm run build`

## Quality gate (the CI mirror)

QA reproduces **every** gate here before opening a PR — a "QA pass" must imply "CI will
pass". The source of truth is the CI workflow file
([`.github/workflows/ci.yml`](../../.github/workflows/ci.yml)): when a job is added or
a command changes there, run the new set, not this list, and update this list in the
same change.

- docs index: `python3 scripts/gen-doc-index --check`
- typecheck:  `npm run check`
- build:      `npm run build`

### Fallbacks

When a tool is genuinely unavailable in the environment, fall back to the nearest
equivalent and **say so explicitly in the report** — never silently skip a gate and
call the run green.

- `astro check` missing → run `npx --yes @astrojs/check` once, then retry `npm run check`.
- `npm ci` fails on lockfile drift → run `npm install` and report the drift; QA bounces
  the bead so the fix lands in the same PR.

## Run the app

How to start the app locally for a smoke check.

- Dev server (hot reload): `npm run dev` — serves at `http://localhost:4321/`.
- Production preview (built site): `npm run build && npm run preview` — serves the same
  static files GitHub Pages will serve, at `http://localhost:4321/`.

Use `preview`, not `dev`, whenever the check has to reflect what Pages actually serves
(base path, image optimization output, etc.).

## UI render check

This project has user-facing UI. QA runs the local render gate for any user-facing bead
before its PR can carry `auto-merge` (see the development lifecycle's review gate).

- Trigger paths: `src/**`, `public/**`, `astro.config.*`, `package.json`, `package-lock.json`
- Harness: **Browser pane preview** — `preview_start` on `npm run preview`, then use
  `mcp__Claude_Browser__resize_window` to switch viewports between desktop 1440×900 and
  mobile 390×844 and take screenshots of every section the bead touches. Compare against
  the design brief's fidelity notes (colors, typography, spacing, exact copy). Attach the
  screenshots to the PR via `scripts/forge upload`.

  Concrete steps QA follows:
  1. `npm ci && npm run build`
  2. `preview_start` the `preview` script.
  3. For each changed section, `navigate` to its anchor and use `resize_window` to switch
     between desktop 1440×900 and mobile 390×844, screenshotting at each viewport.
  4. Visual check against the tokens in `design/README.md` (once the design handoff is
     restored).
  5. `scripts/forge upload <pr> <png>` for each screenshot; embed in the PR body.

  A CSS regression, wrong hex, missing font weight, or crowded hero zoom card is a bounce.
