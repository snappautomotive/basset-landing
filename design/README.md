# Handoff: Basset Landing Page

## Overview
A single-page marketing site for **Basset** — a root-cause-analysis tool for automotive software, a collaboration between **Snapp Automotive** (snappautomotive.io) and **mugo works** (mugoworks.com). The page walks a skeptical engineer from the promise (bug ticket → next step within hours) through the mechanism, the research grounding, failure-mode coverage, honest boundaries ("what Basset is not"), and a single CTA: email to join a paid early-partner evaluation.

Aesthetic: calm, light dev-tool page (Supabase/Tuist family). The product screenshots ARE the argument — everything else stays quiet. The green accent is deliberately taken from the causal-route color inside the product's own UI.

## About the Design Files
The files in `design-reference/` are **design references created in HTML** — a prototype showing intended look and behavior, not production code to copy directly. Recreate this design in the target codebase's existing environment (Next.js/Astro/plain static/etc.) using its established patterns. If no environment exists yet, choose an appropriate one for a mostly-static marketing page (a static-site framework such as Astro or plain HTML/CSS is a fine fit; there is almost no client-side state).

Open `design-reference/Basset Landing.dc.html` in a browser (with the sibling `support.js` and the `../assets` folder in place, e.g. via a local static server from the handoff root) to see the exact reference rendering. Note: the screenshot mounts use a custom `<image-slot>` element in the prototype — in production these are plain `<img>` tags.

## Fidelity
**High-fidelity.** Colors, typography, spacing, copy, and layout are final and approved by the client. Recreate pixel-perfectly. The one open content item: a screenshot for outcome card 03 (visibility-gap view) does not exist yet; the card ships text-only for now.

## Design Tokens

### Colors
| Role | Value |
| --- | --- |
| Page ground | `#fcfcfc` |
| Card / panel surface | `#ffffff` |
| Hairline border | `#e6e8ea` |
| Ink (headings, body) | `#1c1e21` |
| Muted text | `#5f6670` |
| Accent (buttons, solid fills) | `#1d9a5e` |
| Accent hover | `#178350` |
| Accent text/links (AA on white) | `#157347` |
| Link hover | `#0f5c37` |
| Tag fill (neutral) | `#eef0f2` |
| CTA panel fill | `#ecf8f1` |
| CTA panel border | `#cdeeda` |
| CTA panel muted text | `#41564a` |

### Typography
- **Sans**: IBM Plex Sans (400/500/600/700) — all headings and body. Headings weight 600.
- **Mono**: IBM Plex Mono (400/500/600) — kickers, numbers (01/02/03, R1–R3), "not a …" tags, nav byline, footer.
- Load from Google Fonts.

| Style | Spec |
| --- | --- |
| H1 | `clamp(34px, 4.6vw, 52px)` / 1.15 / 600 / letter-spacing -0.025em / max-width 20ch |
| H2 (section) | `clamp(26px, 3vw, 32px)` / 1.28 / 600 / -0.02em |
| H3 (CTA panel) | `clamp(24px, 3vw, 30px)` / 1.28 / 600 / -0.02em |
| Hero subhead | 18px / 29px, muted |
| Body paragraph | 16px / 26px, muted, max-width 60–66ch |
| Emphasis paragraph | 18px / 29px / 600, ink |
| Card title | 17px / 25px / 600 |
| Card body / column body | 14.5px / 23px, muted |
| Kicker | Mono 12.5px, uppercase, letter-spacing 0.12em, color `#157347` |
| Figure caption | 13px / 20px, muted |
| Footer | Mono 13px, muted |

### Spacing & shape
- Content column: max-width 1120px, centered; horizontal padding `clamp(20px, 5vw, 64px)`.
- Section vertical padding: `clamp(48px, 6vw, 72px)`; sections separated by 1px `#e6e8ea` top borders.
- Radii: cards/figures 12px, nested screenshots 8px, buttons 8px, tags 6px, CTA panel 16px, hero zoom card 10px.
- Shadows: hero figure `0 24px 48px -20px rgba(23,24,26,0.18)`; zoom card `0 20px 40px -12px rgba(23,24,26,0.25)`; workspace figure `0 16px 32px -16px rgba(23,24,26,0.14)`; other figures flat (border only).
- `text-wrap: pretty` globally.

## Page Structure (top to bottom)

### 1. Nav bar
White, 1px bottom hairline, padding `16px clamp(20px,5vw,64px)`, flex + 20px gap, wraps on narrow screens.
- Brand: "Basset" 19px/700, ink.
- Byline: mono 12px muted — "by Snapp Automotive × [mugo mark] mugo works" with `assets/mugo-mark.svg` at 18×18 inline.
- Right (flex-pushed): link "Bring us a bug that cost you days. Talk to us." 13.5px/500, `#157347`, no underline, hover `#0f5c37`. Href: `mailto:michel@snappautomotive.io`.

### 2. Hero (left-aligned, never centered)
Padding-top `clamp(48px, 8vw, 88px)`.
- Kicker: "ROOT-CAUSE ANALYSIS FOR AUTOMOTIVE SOFTWARE".
- H1: "We move developers from a bug ticket to the next step. Within hours, not days."
- Subhead: "From a log line to the code that caused it, with the evidence to prove it."
- Primary button: "Talk to us about the evaluation" — solid `#1d9a5e`, white, 15px/600, padding 11px 20px, radius 8px, hover `#178350`. Mailto link.
- Hero figure (margin-top clamp(36px,5vw,56px)): `assets/hero-chain-wide.png` in a 12px-radius bordered white frame with the large shadow, aspect-ratio 3176/2100.
  - **Zoom card** overlapping bottom-right (absolute; right -8px, bottom -48px, width `clamp(220px, 36%, 400px)`): `assets/chain-zoom.png`, aspect 1100/500, 10px radius, border + shadow. This is a crop of the hero image at 1:1 scale (anchor→MARKER segment of the causal route).
  - Caption below (20px margin, cleared of the zoom card by a trailing spacer of `clamp(40px,6vw,72px)` on the hero block): "The causal chain from the symptom back to the code, across process boundaries."

### 3. "The promise" section
Kicker "THE PROMISE"; H2 "With the logs you already have, you always end up with an evidence-backed next step." (max-width 32ch).
Grid of 3 cards — `repeat(auto-fit, minmax(280px, 1fr))`, gap 20px, `align-items: start`; each card: white, 1px border, 12px radius, 24px padding.
- Card 01 — mono "01" green; title "The exact line of code, backed by evidence."; body "A causal chain, as in the view above."
- Card 02 — "02"; title "A set of candidates with confidence levels and evidence."; below it `assets/candidates-view.png` in an 8px-radius bordered frame, aspect 16/10, `object-fit: cover`.
- Card 03 — "03"; title "A highlighted area where visibility fades and further instrumentation is required." (no body, no image yet).
Closing emphasis paragraph (18px/600): "This is slow, painstaking work only your experts could do manually. Now anyone on your team can move from a bug ticket to the next step."

### 4. "The mechanism" section
Kicker "THE MECHANISM"; H2 "How it works, in one pass."
Body (max 66ch): "Basset ingests your source and matches each runtime log line to the exact statement that produced it. No debug symbols needed. It then rebuilds the causal chain across process boundaries from what actually happened in the run, and you walk it clickably into the code. Every step is deterministic and inspectable: no hidden AI invents the answer."
- Figure: `assets/workspace-three-pane.png` (12px radius, border, medium shadow, aspect 3176/2100). Caption: "One workspace: the chain, the code it points to, and the logs that prove it."
- Two-up row (`auto-fit minmax(320px,1fr)`, gap clamp(24px,4vw,48px), centered vertically): `assets/walk-control.png` figure (flat, bordered) + text "You drive the walk. The tool shows its evidence at every hop." (16px/26px muted, max 38ch).

### 5. "The research" section
Kicker "THE RESEARCH"; H2 "Built on how debugging actually works."
Intro: "Before building Basset we studied the research on how developers actually find root causes. Three findings shaped the product:"
One bordered white 12px-radius list container; three rows (grid `56px 1fr`, padding 22px 24px, 1px dividers), mono green labels R1/R2/R3. Each row: bold finding + muted response:
- R1 "Developers spend roughly half of debugging time hunting for the right code, not fixing it." → "So Basset attacks the hunt: from a log line straight to the statement that wrote it."
- R2 "Ranked lists of \"suspicious lines\" fail in practice; developers need the causal story, not a leaderboard." → "So Basset shows a walkable chain of what happened, with evidence at each step."
- R3 "Engineers distrust tools that hide their reasoning." → "So every Basset answer traces back to real code and the real run, and the tool says honestly where its visibility ends."

### 6. Failure-modes section (no kicker)
Two-up row (same responsive grid as §4): left text — H2 "It handles more than one kind of failure." + body "A permission denial traced across processes, or a hard crash: seed the walk from any log record, including a stack frame." Right: `assets/crash-trace.png` figure (flat, bordered), caption "A crash trace: the walk connected the failing record to its cause along a real edge-path."

### 7. "What Basset is not"
H2 "What Basset is not." Three columns (`auto-fit minmax(240px,1fr)`, gap 20px), each: mono tag (13px/600 ink on `#eef0f2`, padding 5px 10px, radius 6px) + muted body 16px below:
- `not a log viewer` → "It starts where viewers stop."
- `not a cloud service` → "It runs where you decide, and your source and logs stay under your control."
- `not an AI oracle` → "Deterministic evidence first, any AI sits on top to explain, never to invent."

### 8. CTA panel
Tinted panel: `#ecf8f1` fill, `#cdeeda` 1px border, 16px radius, padding `clamp(28px,4vw,56px)`, flex wrap, space-between, 32px gap.
- H3 "Bring us a bug that cost you days."
- Body (15.5px/25px, `#41564a`, max 52ch): "Basset is in development and we are taking a small number of early partners. You bring real tickets with logs, we prove it on your hardest cases, and early partners pay roughly half the standard price for the first year."
- Primary button (same style as hero): "Talk to us about the evaluation".

### 9. Footer
White, 1px top hairline, padding `22px clamp(20px,5vw,64px)`, mono 13px muted:
"Basset, a collaboration between [Snapp Automotive](https://www.snappautomotive.io) and [mugo-mark] [mugo works](https://www.mugoworks.com)" — links `#157347`; mugo mark 15×15 inline before "mugo works". **No email address in the footer** (client decision).

## Interactions & Behavior
- All CTAs are `mailto:michel@snappautomotive.io` links (no forms, no backend).
- Link hover: `#157347` → `#0f5c37`. Button hover: `#1d9a5e` → `#178350`. Give buttons/links a visible `:focus-visible` outline per the codebase's a11y conventions.
- No animations or scroll effects in the approved design. If any are added, keep them minimal (e.g. subtle fade-in on figures) — do not add motion that competes with the screenshots.
- Responsive: single content column; grids collapse via `auto-fit` minmax (3→2→1 cards); nav wraps; type scales via the clamps above. The hero zoom card scales down (`clamp(220px,36%,400px)`) and may be hidden below ~480px if it crowds the caption.

## State Management
None. Fully static page.

## Assets (`assets/`)
- `hero-chain-wide.png` — causal-chain graph view (hero). 3176×2100.
- `chain-zoom.png` — 1100×500 crop of the hero (anchor→MARKER segment) for the overlap card.
- `candidates-view.png` — candidate list with confidence levels (card 02). 2906×1866.
- `workspace-three-pane.png` — chain + source + logs workspace. 3778×2132.
- `walk-control.png` — right-click walk-control menu on a node. 2784×1824.
- `crash-trace.png` — crash/stack-frame trace view. 2966×2048.
- `mugo-mark.svg` — official mugo works pine mark, deep-green on light (from the mugo works brand system). Never recolor; on dark grounds request the on-dark variant from mugo works.
- All screenshots are product UI provided by the client; serve them responsively (e.g. `srcset`) — they are large.

Pending from client: outcome-03 screenshot (visibility-gap view), and optionally a hero export without the hover tooltip.

## Files
- `design-reference/Basset Landing.dc.html` — the approved hi-fi reference page.
- `design-reference/support.js`, `design-reference/image-slot.js` — prototype runtime only; not part of the design. Needed only to open the reference file locally.
- `assets/` — production assets listed above.
