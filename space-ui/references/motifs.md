# Sci-Fi UI Motifs — Atmosphere Primitives

A catalog of small, reusable, accent-themed details that make a UI read as futuristic.
All use the tokens from `color-system.md` (never a raw hex). Use them **sparingly** — one
or two per view. Restraint is the aesthetic; a screen covered in brackets and sweeps looks
like a toy, not a console. Every animated motif is wrapped in
`@media (prefers-reduced-motion: no-preference)`.

Verified: the clip-path, HUD-frame, segmented-meter, conic-sweep, and hazard recipes were
rendered in Chrome and confirmed.

---

## 1. Corner brackets / HUD frame

The L-shaped marks framing a region — a viewfinder/reticle cue that says "this is a
viewport looking at something." The signature space-ui motif.

**Compact (2 pseudo-elements, opposite corners):**
```css
.hud-frame { position: relative; }
.hud-frame::before, .hud-frame::after {
  content: ''; position: absolute; width: 16px; height: 16px;
  border: 1px solid var(--accent); opacity: .7; pointer-events: none;
}
.hud-frame::before { top: 4px; left: 4px; border-right: 0; border-bottom: 0; }
.hud-frame::after  { bottom: 4px; right: 4px; border-left: 0; border-top: 0; }
```

**Full (all four corners, one span each) — when you want the complete frame:**
```html
<div class="framed">
  <span class="fc tl"></span><span class="fc tr"></span>
  <span class="fc bl"></span><span class="fc br"></span>
  … content …
</div>
```
```css
.framed { position: relative; }
.fc { position: absolute; width: 14px; height: 14px; border: 1px solid var(--accent);
  opacity: .7; pointer-events: none; }
.fc.tl { top: 4px; left: 4px;  border-right: 0; border-bottom: 0; }
.fc.tr { top: 4px; right: 4px; border-left: 0;  border-bottom: 0; }
.fc.bl { bottom: 4px; left: 4px;  border-right: 0; border-top: 0; }
.fc.br { bottom: 4px; right: 4px; border-left: 0;  border-top: 0; }
```
Optional label chip in a corner (e.g. `GRAPH · VIEWPORT`) in the display font, `--dim`,
wide letter-spacing — reinforces the "instrument readout" feel.

## 2. Panel corners — square vs notched (configurable)

Panels are the workhorse surface. Corners are **sharp by default** (sci-fi has no rounded
corners) and can be switched to a **notched / chamfered** silhouette via one variable —
mirroring the "one knob" philosophy of the color system. Set the corner style once on a
root/container and every panel follows.

```css
:root {
  --notch: 0;        /* 0 = square (sharp) · e.g. 14px = notched/chamfered */
  --panel-border: 1.5px;
}

/* Base panel — SHARP corners + luminous border.
   The border uses --accent-border (visible, not the faint --line) plus a soft
   outer glow and a faint inner accent line, so panels read as lit rather than
   muted gray. The inner line is the tasteful touch — keep it subtle (~5%). */
.panel {
  position: relative;
  background: var(--panel);
  border: 1px solid var(--accent-border);
  border-radius: 0;               /* sci-fi = sharp; do not round */
  padding: 1.25rem;
  box-shadow:
    0 0 12px -2px var(--accent-glow),
    inset 0 0 0 1px color-mix(in srgb, var(--accent) 5%, transparent);
}
```

**Notched variant** — chamfers the **top-left + bottom-right** diagonal corners (the
console-panel silhouette) with a crisp border that follows the cut. Because `clip-path`
would clip a normal border unevenly, the border is drawn as a two-layer sandwich: an
accent-filled `::before` (the border) with the dark fill `::after` inset by the border
width. Verified in Chrome.

```css
.panel.notched {
  background: transparent; border: 0;
  filter: drop-shadow(0 0 6px var(--accent-glow));   /* optional outer glow */
}
.panel.notched::before, .panel.notched::after {
  content: ''; position: absolute; inset: 0;
  /* two diagonal corners cut: TL + BR (TR + BL stay square) */
  clip-path: polygon(
    var(--notch) 0, 100% 0,
    100% calc(100% - var(--notch)), calc(100% - var(--notch)) 100%,
    0 100%, 0 var(--notch));
}
.panel.notched::before { background: var(--accent); }               /* glowing border */
.panel.notched::after  { background: var(--panel); inset: var(--panel-border); } /* fill */
.panel.notched > * { position: relative; z-index: 1; }              /* keep content above */
```

Variants:
- **Subtle** (no glow): `.panel.notched::before { background: var(--line); }` and drop the
  `filter`.
- **All four corners** (full chamfer, for hero/feature panels): use the 8-point polygon
  `polygon(var(--notch) 0, calc(100% - var(--notch)) 0, 100% var(--notch),
  100% calc(100% - var(--notch)), calc(100% - var(--notch)) 100%, var(--notch) 100%,
  0 calc(100% - var(--notch)), 0 var(--notch))`.
- **Square** is just `--notch: 0` (or don't add `.notched`) — a sharp-cornered panel.

Keep one corner style per UI for consistency; expose `--notch` so the user can flip the
whole interface between square and notched.

## 3. Eyebrow / kicker label

Small uppercase label above a heading — the "section designation."
```css
.eyebrow { font-family: var(--display); font-size: .68rem; letter-spacing: .28em;
  text-transform: uppercase; color: var(--accent); }
/* Optional as a bordered chip with a live dot: */
.eyebrow-chip { display: inline-flex; align-items: center; gap: .6rem; padding: .4rem .8rem;
  border: 1px solid var(--accent-line); background: var(--accent-soft); border-radius: 0; }
.eyebrow-chip::before { content: ''; width: 6px; height: 6px; border-radius: 50%;
  background: var(--accent); box-shadow: 0 0 10px var(--accent); }
```

## 4. Coordinate / telemetry readout

The `28°33'N` style corner text — fake instrument data that sells the HUD.
```css
.readout-hud { font-family: var(--display); font-size: .7rem; letter-spacing: .22em;
  line-height: 1.8; color: var(--dim); }
.readout-hud .label { color: var(--accent); font-size: .65rem; }
.readout-hud span { display: block; font-variant-numeric: tabular-nums; }
```

## 5. Glow divider (luminous hairline)

A horizontal rule that fades in from the edges with an accent core.
```css
.divider { height: 1px; border: 0;
  background: linear-gradient(90deg, transparent, var(--accent-ring), transparent); }
```

## 6. Segmented meter / LED bar

Progress as discrete lit segments (more "instrument" than a smooth bar).
```html
<div class="seg" role="progressbar" aria-valuenow="3" aria-valuemin="0" aria-valuemax="8">
  <i class="on"></i><i class="on"></i><i class="on"></i><i></i><i></i><i></i><i></i><i></i>
</div>
```
```css
.seg { display: flex; gap: 3px; height: 14px; }
.seg i { flex: 1; background: var(--accent-line); }
.seg i.on { background: var(--accent); box-shadow: 0 0 6px var(--accent-glow); }
```

## 7. Scan sweep (radar beam)

A rotating conic wedge — great for "loading / scanning / live" states.
```css
.sweep { position: relative; border-radius: 50%; border: 1px solid var(--line); overflow: hidden; }
.sweep::after { content: ''; position: absolute; inset: 0;
  background: conic-gradient(from 0deg, transparent 0deg,
    var(--accent-soft) 40deg, transparent 60deg); }
@media (prefers-reduced-motion: no-preference) {
  @keyframes sweep { to { transform: rotate(360deg); } }
  .sweep::after { animation: sweep 4s linear infinite; }
}
```
Line-scan variant: a thin horizontal bar translating top→bottom over a panel
(`linear-gradient` beam + `translateY` keyframes), same reduced-motion guard.

## 8. Targeting reticle (dashed rotating ring)

A dashed circle that slowly rotates — orbital/scanner cue. (Used in dijkstra's rocket stage.)
```css
.reticle { position: relative; width: 120px; height: 120px; }
.reticle::before { content: ''; position: absolute; inset: 0; border-radius: 50%;
  border: 1px solid var(--accent-line); }
.reticle::after { content: ''; position: absolute; inset: -10px; border-radius: 50%;
  border: 1px dashed var(--accent-line); }
@media (prefers-reduced-motion: no-preference) {
  @keyframes spin { to { transform: rotate(360deg); } }
  .reticle::after { animation: spin 30s linear infinite; }
}
```

## 9. Hazard stripes

Diagonal caution bars for warning edges / restricted zones. Use with `--warn`/`--crit`
for real warnings; with `--accent-soft` as decoration.
```css
.hazard { height: 16px;
  background: repeating-linear-gradient(45deg,
    var(--warn) 0 8px, transparent 8px 16px); opacity: .5; }
```

## 10. Terminal caret (blinking cursor)

For command inputs / typing effects.
```css
.caret::after { content: ''; display: inline-block; width: .5ch; height: 1em;
  background: var(--accent); vertical-align: text-bottom; margin-left: 2px; }
@media (prefers-reduced-motion: no-preference) {
  @keyframes blink { 0%,50% { opacity: 1 } 50.01%,100% { opacity: 0 } }
  .caret::after { animation: blink 1s step-end infinite; }
}
```

## 11. Vignette glow

A soft radial accent glow behind hero/focal content — depth without an image.
```css
.glow-bg { position: relative; }
.glow-bg::before { content: ''; position: absolute; inset: 0; pointer-events: none; z-index: 0;
  background: radial-gradient(ellipse at 50% 30%, var(--accent-soft), transparent 60%); }
.glow-bg > * { position: relative; z-index: 1; }
```

## 12. Starfield (CSS-only, no image)

Layered `radial-gradient` dots for a cheap deep-space backdrop (used in dijkstra's "work" section).
```css
.stars { background-image:
  radial-gradient(1px 1px at 20% 30%, rgba(255,255,255,.5), transparent),
  radial-gradient(1px 1px at 70% 60%, var(--accent-glow), transparent),
  radial-gradient(1px 1px at 40% 80%, rgba(255,255,255,.4), transparent),
  radial-gradient(1px 1px at 85% 25%, rgba(255,255,255,.3), transparent);
  background-repeat: no-repeat; }
```

---

## Usage guidance

- **Landing:** HUD frame + vignette glow + eyebrow chip around the hero; maybe a starfield
  in a secondary section.
- **Forms:** eyebrow labels; a glow divider between groups; caret on a terminal-style field.
- **Dashboards:** notched/framed panels; segmented meters for KPIs; glow dividers.
- **Control-center:** framed panels, scan sweep on live regions, reticle on a radar/topology
  widget, hazard stripes + `--crit` on alert zones, telemetry readouts everywhere.
- **Budget:** ~1–2 decorative motifs per view. Decorative-only elements get
  `pointer-events: none` and `aria-hidden="true"`; functional ones (meters, progress) get
  proper roles/labels. Keep animations off large full-viewport surfaces (paint cost) and
  always behind the reduced-motion guard.
