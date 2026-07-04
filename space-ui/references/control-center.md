# Mode: Control Center

A monitoring / ops console / command wall — a dense dashboard whose job is live
situational awareness: status indicators, telemetry, alerts, and quick actions. This is
the most "sci-fi HUD" mode. Assume **live, interactive data** → React by default
(`references/react.md`); read `dashboards.md` first (this builds on it).

## Intent

Optimize for **at-a-glance state**, not marketing polish: high information density,
strong status color-coding, and motion reserved for things that demand attention (an
alert pulse), not decoration. A calm control center is mostly `--ok`/idle; color and
motion spike only on `--warn`/`--crit`.

## Layout

A fixed full-height grid — no page scroll for the primary wall; panels scroll internally.

```css
.cc { display: grid; grid-template-columns: repeat(12, 1fr); grid-auto-rows: minmax(120px, auto);
  gap: 1rem; height: 100vh; padding: 1rem; }
.cc-header { grid-column: 1 / -1; display: flex; align-items: center; gap: 1rem; }
.cc-panel { overflow: auto; } /* individual panels scroll, not the page */
```
Give panels explicit spans (`grid-column: span N; grid-row: span N`). Common regions:
system map / topology, live metric charts, event log stream, alert list, quick-action bar.

## Status indicator (the core primitive)

```html
<span class="status ok"><span class="dot"></span>NOMINAL</span>
```
```css
.status { display: inline-flex; align-items: center; gap: .5rem;
  font-family: var(--display); font-size: .68rem; letter-spacing: .22em; text-transform: uppercase; }
.status .dot { width: 8px; height: 8px; border-radius: 50%; }
.status.ok   { color: var(--ok);   } .status.ok   .dot { background: var(--ok);   box-shadow: 0 0 8px var(--ok); }
.status.warn { color: var(--warn); } .status.warn .dot { background: var(--warn); box-shadow: 0 0 8px var(--warn); }
.status.crit { color: var(--crit); } .status.crit .dot { background: var(--crit); box-shadow: 0 0 8px var(--crit); }
.status.idle { color: var(--dim);  } .status.idle .dot { background: var(--dim); }
/* Only critical pulses — draws the eye. */
@media (prefers-reduced-motion: no-preference) {
  @keyframes pulse { 0%,100%{opacity:1} 50%{opacity:.35} }
  .status.crit .dot { animation: pulse 1.2s ease-in-out infinite; }
}
```

## Telemetry readout

Big monospaced-feel numbers in the display font, with unit and trend:

```css
.readout { display: grid; gap: .2rem; }
.readout-value { font-family: var(--display); font-size: clamp(1.6rem, 3vw, 2.6rem); color: #fff; line-height: 1; font-variant-numeric: tabular-nums; }
.readout-unit { color: var(--dim); font-size: .7rem; letter-spacing: .18em; text-transform: uppercase; }
.readout.crit .readout-value { color: var(--crit); text-shadow: 0 0 20px var(--crit); }
```

## Gauge / meter (SVG ring)

```html
<svg viewBox="0 0 40 40" class="gauge" role="img" aria-label="CPU load 72%">
  <circle cx="20" cy="20" r="16" fill="none" stroke="var(--line)" stroke-width="3"/>
  <circle cx="20" cy="20" r="16" fill="none" stroke="var(--accent)" stroke-width="3"
    stroke-linecap="round" stroke-dasharray="100.53" stroke-dashoffset="28.15"
    transform="rotate(-90 20 20)"/>
</svg>
```

**Compute the offset — don't hand-guess it.** For radius `r` and a percentage
`pct` (0–100):
- Circumference `C = 2 · π · r`. For `r = 16` → `C ≈ 100.53`.
- Set `stroke-dasharray = C` and `stroke-dashoffset = C · (1 − pct/100)`.
  - 72% of `r=16` → offset `= 100.53 · 0.28 ≈ 28.15`.
- `rotate(-90 …)` starts the arc at 12 o'clock; `stroke-linecap="round"` gives the
  rounded tip.

In React/JS, drive it live from state so you never hardcode:
```js
const C = 2 * Math.PI * r;                 // r = 16
const dashoffset = C * (1 - pct / 100);
```
Switch the stroke to `var(--warn)` / `var(--crit)` past threshold values.

## Live event log

A scrolling stream; newest on top. Each row: timestamp (`--dim`), severity chip, message.
Color the left border by severity. Cap DOM rows (windowing) for long-running streams.

```css
.log-row { display: grid; grid-template-columns: auto auto 1fr; gap: .6rem; padding: .4rem .6rem;
  border-left: 2px solid var(--line); font-size: .82rem; }
.log-row.warn { border-left-color: var(--warn); } .log-row.crit { border-left-color: var(--crit); }
.log-time { color: var(--dim); font-variant-numeric: tabular-nums; }
```

## Live data (React)

- Poll/subscribe (WebSocket/SSE) in an effect; keep a bounded buffer in state.
- Announce new critical alerts to AT with an `aria-live="assertive"` region.
- Throttle high-frequency updates (rAF or a fixed tick) so React doesn't thrash.
- Respect `prefers-reduced-motion`: no pulsing/sweeping for those users; still update text.
- See `references/react.md` for the token wiring and a status-driven component pattern.

## Checklist (control-center)

- Status colors are the derived `--ok/--warn/--crit` tokens; motion only on `--crit`.
- Fixed-height wall; panels scroll internally, not the page.
- Gauges/charts have `role="img"` + `aria-label`; live regions announce critical changes.
- Reduced-motion respected. Density high but readable (tabular numerals, clear hierarchy).
