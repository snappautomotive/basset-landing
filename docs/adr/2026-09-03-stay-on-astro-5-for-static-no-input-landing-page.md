---
title: ADR-0001 Stay on Astro 5.x for the static no-input landing page
summary: Why the landing page stays on astro@^5.18.0 despite eight high-severity npm-audit advisories — none is reachable on a static site with no user-controlled input, and the Astro 7 upgrade currently breaks the build via an unrelated Rolldown tsconfig-resolver regression.
updated: 2026-09-03
status: accepted
related:
  - package.json
  - astro.config.mjs
  - src/pages/index.astro
  - src/layouts/BaseLayout.astro
---

# ADR-0001 — Stay on Astro 5.x for the static no-input landing page

## Status

Accepted — 2026-09-03.

## Context

`npm audit --omit=dev` on `astro@5.18.2` reports three vulnerable packages (eight
high-severity Astro advisories, one high-severity sharp advisory, one low-severity
esbuild advisory) with the fix gated on a breaking upgrade to `astro@7.2.10`:

| # | Advisory | Package | Severity | Trigger surface |
|---|----------|---------|----------|------------------|
| 1 | [GHSA-j687-52p2-xcff](https://github.com/advisories/GHSA-j687-52p2-xcff) | astro | high | Attacker-controlled value passed into a `define:vars` directive |
| 2 | [GHSA-xr5h-phrj-8vxv](https://github.com/advisories/GHSA-xr5h-phrj-8vxv) | astro | high | Server island encrypted parameters replayed across an SSR request |
| 3 | [GHSA-jrpj-wcv7-9fh9](https://github.com/advisories/GHSA-jrpj-wcv7-9fh9) | astro | high | Attacker-controlled object keys spread into an HTML element's attributes |
| 4 | [GHSA-f48w-9m4c-m7f5](https://github.com/advisories/GHSA-f48w-9m4c-m7f5) | astro | high | Same class as #3 (`renderHTMLElement` spread-attribute names) |
| 5 | [GHSA-7pw4-f3q4-r2p2](https://github.com/advisories/GHSA-7pw4-f3q4-r2p2) | astro | high | Attacker-controlled `transition:*` directive value on a hydrated island |
| 6 | [GHSA-4g3v-8h47-v7g6](https://github.com/advisories/GHSA-4g3v-8h47-v7g6) | astro | high | Attacker-controlled View-Transition animation property |
| 7 | [GHSA-2pvr-wf23-7pc7](https://github.com/advisories/GHSA-2pvr-wf23-7pc7) | astro | high | Attacker-controlled `Host` header reaches the prerendered error page fetch (SSR only) |
| 8 | [GHSA-8hv8-536x-4wqp](https://github.com/advisories/GHSA-8hv8-536x-4wqp) | astro | high | Attacker-controlled slot name reflected into the page |
| 9 | [GHSA-f88m-g3jw-g9cj](https://github.com/advisories/GHSA-f88m-g3jw-g9cj) | sharp (libvips) | high | Attacker-supplied image processed through sharp/libvips |
| 10 | [GHSA-g7r4-m6w7-qqqr](https://github.com/advisories/GHSA-g7r4-m6w7-qqqr) | esbuild | low | Dev-server running on Windows exposed to the network |

Every advisory in the list requires an attacker to control a piece of input that flows
into the vulnerable code path. The landing page has no such input surface:

- Deploy target is **GitHub Pages** — the entire site is prerendered to `dist/` at
  build time. There is no server, no SSR, no runtime `Astro.request` / `Astro.url`,
  and no host header ever reaches Astro after deploy.
- The single page (`src/pages/index.astro`) composes eight authored components. There
  are no forms, no `<input>` elements, no query-parameter reading, no client-side
  routing, no client-side state, and no JavaScript that runs against user input.
- Every call to action is a `mailto:` link — see `Hero.astro`, `Nav.astro`, and
  `CtaPanel.astro`. The address is a compile-time constant.
- The `<Image>` component is only used to emit responsive `<picture>` markup for four
  design-team PNGs imported from `design/assets/*.png` at build time. sharp only ever
  processes those four fixed inputs; the built site contains only the resulting
  static assets.
- No component uses `define:vars`, `<ViewTransitions />` / `ClientRouter`,
  `transition:*` directives, server islands (`server:defer`), spread props into HTML
  elements, dynamic `<slot name={...}>`, or any `client:*` hydration directive
  (`grep`ed under `src/`).
- The site is authored and reviewed on macOS/Linux; the esbuild Windows dev-server
  advisory has no reachable target.

The applicable-input analysis maps 1:1 to the advisory table above:

| # | Applies here? | Why |
|---|---------------|-----|
| 1 | No | `grep -R 'define:vars' src/` returns nothing. |
| 2 | No | No server output; `output: "static"` (Astro 5 default) and no `server:defer`. |
| 3 | No | No `{...props}` on any HTML element. All props are named. |
| 4 | No | Same reason as #3. |
| 5 | No | No `transition:*` directive used. No client hydration. |
| 6 | No | No `<ViewTransitions />` / `<ClientRouter />` mounted. |
| 7 | No | Fully static output — Astro serves nothing at runtime. |
| 8 | No | The only `<slot />` is unnamed (`BaseLayout.astro`). Nothing dynamic. |
| 9 | No | sharp only processes four fixed PNGs shipped in `design/assets/` at build time. |
| 10 | No | Windows-only, dev-server-only. Development is macOS; CI builds on Linux without running the dev server. |

Because none of the vulnerable code paths is reachable, all ten advisories are
**inapplicable** on this deployment.

## Options considered

### Path A — upgrade `astro` to `^7.2.10`

Attempted first. `npm install` runs cleanly and Astro 7 clears every advisory
(`found 0 vulnerabilities`), but the production build fails with:

```
[TSCONFIG_ERROR] Failed to load tsconfig 'astro/tsconfigs/strict': Tsconfig not found
```

The failure is a Rolldown resolver regression (Vite 8 / Rolldown 1.x tsconfig
walking) that ignores this worktree's own `tsconfig.json` and picks up an unrelated
sibling worktree's file whose `extends` cannot be resolved from the current
`node_modules/`. Disabling `resolve.tsconfigPaths` in `astro.config.mjs` moves the
error out of the `rolldown:vite-resolve` plugin but Rolldown still resolves the
wrong tsconfig chain from a deeper transform step. Related upstream issues that
document the same class of Rolldown resolver problem in this release window:
[vitejs/vite#21856](https://github.com/vitejs/vite/issues/21856),
[rolldown/rolldown#8732](https://github.com/rolldown/rolldown/issues/8732),
[vitejs/vite#22322](https://github.com/vitejs/vite/issues/22322).

Spending unbounded time working around a resolver bug — for advisories that are not
reachable on this page — trades a real, current, working build for a speculative
future-security posture that is already satisfied by the site's shape.

### Path B — stay on `astro@^5.18.0`, document the reasoning

Adopted. The site's threat model has no attacker-controlled input on any of the
vulnerable code paths, and the current build is green. The two long-lived
non-Astro advisories (sharp/libvips, esbuild) are similarly unreachable here.

### Path C — file a follow-up upgrade bead

Not filed. When a later Astro 7.x patch resolves the Rolldown tsconfig resolver bug
(or when we adopt a version that pins its own Rolldown/Vite pair over the failing
range), we will revisit — but re-verify the reachability table first, since the
answer here is "no advisory applies" rather than "the fix is expensive". A future
bead should reference this ADR and re-run `npm audit --omit=dev` against the surface
described above.

## Decision

Stay on `astro@^5.18.0` for the landing page. Do **not** upgrade to Astro 7 solely
for advisory closure. Reachability of every listed advisory is nil under the
site's current architecture (static build, no user input, no SSR, no hydration,
no dynamic image processing).

## Consequences

- `npm audit --omit=dev` will continue to report the same three advisory groups.
  Any CI or human review that reads audit output must consult this ADR before
  treating those advisories as blocking.
- The next PR that introduces any of the following triggers a full re-evaluation
  and **supersedes this ADR**:
  - A form or any user-input surface (`<input>`, `<textarea>`, `<form>`, query
    parameters, cookies).
  - `define:vars`, `<ViewTransitions />` / `<ClientRouter />`, `transition:*`
    directives, server islands / `server:defer`, or any `client:*` hydration
    directive.
  - Spread props into HTML elements, dynamic slot names, or SSR output.
  - Runtime image processing of any input not shipped in `design/assets/`.
- The next time we touch dependencies for another reason, we retry the Astro 7
  upgrade; if the Rolldown resolver regression is fixed, that PR carries the
  upgrade and marks this ADR `superseded`.

## References

- `package.json` — `astro@^5.18.0` pin.
- `astro.config.mjs` — the site config (no SSR, no adapter, static output).
- `src/pages/index.astro` — the single page and its component tree.
- `src/layouts/BaseLayout.astro` — the only `<slot />` (unnamed, no dynamic props).
- Astro upgrade guides:
  [v6](https://docs.astro.build/en/guides/upgrade-to/v6/),
  [v7](https://docs.astro.build/en/guides/upgrade-to/v7/).
