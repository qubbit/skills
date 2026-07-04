---
name: space-ui
description: Build futuristic / sci-fi / space-age / cyberpunk / high-tech user interfaces — landing pages, forms, dashboards, and control centers — as raw HTML or React components, all themed from a single configurable accent color. Use whenever the user wants a "space age", "futuristic", "sci-fi", "cyberpunk", "hi-tech", or "cinematic" look for: a landing / hero / splash page; a form or input UI; an admin dashboard, analytics dashboard, data panel, or stat grid; or a monitoring "control center" / command console / ops console with live data and status indicators. Also triggers on "make this look futuristic", "sci-fi dashboard", "control center UI", "neon admin panel", and requests for near-perfect Lighthouse scores on a striking static page. Supports raw HTML (zero-JS, Lighthouse-100 for the landing mode) and React component output. All accent color variants are derived from one chosen color via CSS color math.
---

# Space UI — Futuristic Interface Builder

Build interfaces in a **cinematic-futurism** design language — deep space blacks, a
single luminous accent, thin glowing borders, fine grids, and precise geometric type.
The same design system spans four artifact types (modes) and two output targets.

## Design language (applies to every mode)

- **Aesthetic:** the opening frame of a sci-fi film — vast, dark, precise, luminous.
  Restraint is power: one accent, sparse glow, lots of near-black space.
- **Color:** ONE author-chosen accent; all other accent variants are *derived by CSS
  color math* (never hand-picked). This is mandatory — see `references/color-system.md`
  and always start by dropping its `:root` token block in. Default accent: cyan `#00f0ff`.
- **Type:** exactly TWO Google Fonts — a geometric/display face for headings/labels
  (Chakra Petch, Orbitron, Rajdhani, Exo 2, Audiowide, Bruno Ace) and a clean sans for
  body (Sora, Outfit, Plus Jakarta Sans, Manrope). Never Inter, Roboto, or Arial.
  Uppercase + wide letter-spacing (`0.16em`–`0.32em`) on labels/eyebrows is the tell.
- **Corners:** SHARP by default — `border-radius: 0` on panels, cards, inputs, and
  buttons. Sci-fi UIs are angular; rounded corners read as consumer/friendly, not
  cinematic. For visual interest, chamfer (notch) corners via the configurable `--notch`
  variable (see `references/motifs.md`) — never round them. Small radii (≤2px) are the
  most rounding ever acceptable, and only on tiny chips.
- **Atmosphere:** scanline overlays, masked grid patterns, soft radial glows, 1px
  accent-tinted borders (`--accent-line`), glow shadows (`--accent-glow`) used sparingly.
- **Motion:** subtle; always wrapped in `@media (prefers-reduced-motion: no-preference)`.
- **A11y baseline (all modes):** semantic landmarks, one `<h1>` on pages, visible
  `:focus-visible` rings (`--accent-ring`), 44px touch targets, WCAG AA contrast,
  labels tied to inputs.

## Step 1 — Pick the mode

Detect from the request; if genuinely ambiguous, ask. Then read that mode's reference
file for detailed patterns.

| Mode | Use when the user wants… | Reference |
|------|--------------------------|-----------|
| **landing** | a landing / hero / splash / marketing page; "fast page", Lighthouse scores | `references/landing.md` |
| **form** | inputs, sign-in/up, settings, multi-step forms, any data-entry UI | `references/forms.md` |
| **dashboard** | admin/analytics dashboard, stat grid, data tables, charts | `references/dashboards.md` |
| **control-center** | monitoring/ops console, command center, live status wall | `references/control-center.md` |

Modes compose — a dashboard may embed forms; a control-center is a dense dashboard with
live status. Read the primary mode's file plus any it depends on.

## Step 2 — Pick the output target

Ask if unstated (unless the project context makes it obvious, e.g. an existing repo).

- **Raw HTML** — self-contained `.html` with inlined CSS. The **landing** mode in HTML is
  held to the strict zero-JS, Lighthouse-95–100 bar (see below +
  `references/lighthouse-checklist.md`). Other modes in HTML may use minimal vanilla JS
  where interactivity is required.
- **React** — `.jsx`/`.tsx` component files sharing the same CSS token layer (a CSS module
  or a single `theme.css`). See `references/react.md`. React mode optimizes for component
  reuse, correct state, and accessibility — NOT the zero-JS Lighthouse bar. Dashboards and
  control-centers assume live/interactive data and are React-leaning by default.

**Color is configurable in BOTH targets** via the `:root` (HTML) / `theme.css` (React)
token block. If the user names an accent, set `--accent`; otherwise default cyan and tell
them the one line to change.

## Step 3 — Always start with the color tokens

Before writing any component, drop in the `:root` token block from
`references/color-system.md`. Set `--accent` to the user's color (or cyan). Every color
you use thereafter references a token (`var(--accent-bright)`, `var(--accent-line)`, …).
Never write a raw accent hex outside that block. This is what makes the whole UI re-theme
from a single value.

## Shared atmosphere CSS (mode-agnostic)

```css
/* Scanline overlay */
.scanlines::after {
  content: ''; position: absolute; inset: 0; pointer-events: none;
  background: repeating-linear-gradient(to bottom,
    transparent, transparent 3px,
    color-mix(in srgb, var(--accent) 2%, transparent) 3px,
    color-mix(in srgb, var(--accent) 2%, transparent) 4px);
}
/* Masked grid backdrop */
.grid-bg {
  background-image:
    linear-gradient(var(--accent-line) 1px, transparent 1px),
    linear-gradient(90deg, var(--accent-line) 1px, transparent 1px);
  background-size: 72px 72px;
  -webkit-mask-image: radial-gradient(ellipse at center, #000 15%, transparent 75%);
          mask-image: radial-gradient(ellipse at center, #000 15%, transparent 75%);
}
/* Panel — SHARP corners (border-radius: 0; sci-fi UIs never round — chamfer via
   --notch instead, see motifs.md). The border is a luminous accent line
   (--accent-border) with a soft outer glow and a faint inner accent line, so
   panels read as lit rather than muted gray. */
.panel {
  background: linear-gradient(180deg, var(--panel), #070812);
  border: 1px solid var(--accent-border); border-radius: 0; padding: 1.25rem;
  box-shadow:
    0 0 12px -2px var(--accent-glow),                      /* soft outer glow */
    inset 0 0 0 1px color-mix(in srgb, var(--accent) 5%, transparent); /* faint inner line */
}
/* Eyebrow / section label */
.eyebrow {
  font-family: var(--display); font-size: 0.68rem; letter-spacing: 0.28em;
  text-transform: uppercase; color: var(--accent);
}
```

**Sci-fi motifs** — the signature details (HUD **corner brackets**, notched/clipped panels,
scan sweeps, targeting reticles, segmented meters, telemetry readouts, glow dividers,
hazard stripes, terminal caret, starfields) live in `references/motifs.md`. Read it and
use 1–2 per view to establish the aesthetic. All are accent-token-themed and
reduced-motion-guarded.

Full component recipes (buttons, inputs, cards, tables, status lights, gauges) live in
the mode reference files, all built on these tokens.

## Static-HTML landing mode — the strict bar

When (and only when) building a **landing page as raw HTML**, hold the Lighthouse 95–100
standard across all four categories. The full rules are in
`references/lighthouse-checklist.md`; the essentials:

- Zero `<script>` tags. Inline all CSS in one `<head>` `<style>`. No external CSS.
- Only external resource is Google Fonts (2 preconnects + one `display=swap` link).
- Hero `<img>` has explicit `width`/`height`, `fetchpriority="high"`, `decoding="async"`,
  never `loading="lazy"`. Meaningful `alt`.
- Provide the static color fallbacks from `color-system.md` so old engines still theme.
- `<!DOCTYPE html>` line 1, `<meta charset="UTF-8">` first in head, one `<h1>`, skip link
  first in body, semantic landmarks, `<title>` < 60 chars, `<meta name="description">` < 160.

Other modes do not chase Lighthouse-100 (they're app UIs) but keep the a11y baseline.

## Output

- **HTML:** one self-contained `.html` per page. Save to the user's requested path (or
  `/mnt/user-data/outputs/` when no path is given) and present it.
- **React:** component file(s) + a shared `theme.css` (or CSS module) holding the tokens.
  Keep components focused; colocate styles per the project's convention.
- State assumptions (accent chosen, mode, target) briefly so the user can adjust.

## Pre-delivery checklist

1. The `:root`/`theme.css` token block is present and `--accent` is set to the user's color.
2. No raw accent hex anywhere except the token block — everything uses `var(--accent-*)`.
3. Two Google Fonts only; display face on labels/headings, sans on body.
4. `:focus-visible` rings on every interactive element; 44px targets; labels ↔ inputs.
5. Animations wrapped in `prefers-reduced-motion`.
6. Mode-specific checklist from the reference file satisfied.
7. **Landing + HTML only:** the full `references/lighthouse-checklist.md` passes.
