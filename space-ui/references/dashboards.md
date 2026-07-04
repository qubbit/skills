# Mode: Dashboards

Admin / analytics dashboards — stat cards, data grids, tables, and lightweight charts,
in the space-ui language. Assume live/interactive data → React-leaning by default
(`references/react.md`), but the CSS below is framework-agnostic.

## Layout shell

```
┌ sidebar ┬─ topbar (title · search · user) ──────────────┐
│ nav     ├─ stat row (KPI cards) ───────────────────────┤
│         ├─ main grid (charts · tables · panels) ───────┤
└─────────┴───────────────────────────────────────────────┘
```
```css
.dash { display: grid; grid-template-columns: 240px 1fr; min-height: 100vh; }
.dash-main { display: grid; grid-template-rows: auto auto 1fr; gap: 1.25rem; padding: 1.5rem; }
.stat-row { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 1rem; }
.dash-grid { display: grid; grid-template-columns: repeat(12, 1fr); gap: 1.25rem; }
.col-4 { grid-column: span 4; } .col-6 { grid-column: span 6; } .col-8 { grid-column: span 8; }
@media (max-width: 900px) {
  .dash { grid-template-columns: 1fr; }        /* collapse sidebar to a drawer */
  .dash-grid > * { grid-column: 1 / -1; }
}
```

## Sidebar nav

```css
.sidebar { background: linear-gradient(180deg, var(--panel), var(--bg)); border-right: 1px solid var(--line); padding: 1.25rem; }
.nav-item { display: flex; align-items: center; gap: .7rem; padding: .7rem .8rem; border-radius: 0;
  font-family: var(--display); font-size: .8rem; letter-spacing: .06em; text-transform: uppercase;
  color: var(--muted); text-decoration: none; }
.nav-item:hover { color: var(--accent); background: var(--accent-soft); }
.nav-item[aria-current=page] { color: var(--accent); background: var(--accent-fill);
  box-shadow: inset 2px 0 0 var(--accent); }
```

## KPI / stat card

```html
<article class="stat panel">
  <span class="eyebrow">Active Nodes</span>
  <strong class="stat-value">1,284</strong>
  <span class="stat-delta up">▲ 4.2%</span>
</article>
```
```css
.stat { display: grid; gap: .4rem; }
.stat-value { font-family: var(--display); font-size: 2rem; color: #fff; line-height: 1; }
.stat-delta { font-size: .78rem; font-family: var(--display); letter-spacing: .1em; }
.stat-delta.up { color: var(--ok); } .stat-delta.down { color: var(--crit); }
/* top hairline that lights on hover */
.stat { position: relative; overflow: hidden; }
.stat::after { content: ''; position: absolute; top: 0; left: 0; width: 0; height: 1px; background: var(--accent); }
@media (prefers-reduced-motion: no-preference){ .stat{ transition: none } .stat:hover::after{ width: 100%; transition: width .4s ease } }
```

## Data table

```css
.table { width: 100%; border-collapse: collapse; font-size: .9rem; }
.table thead th { font-family: var(--display); font-size: .64rem; letter-spacing: .16em;
  text-transform: uppercase; color: var(--accent); text-align: left; padding: .6rem .75rem;
  background: var(--accent-soft); }
.table td { padding: .55rem .75rem; color: var(--muted); border-bottom: 1px solid var(--line); }
.table tbody tr:hover td { color: var(--ink); background: var(--accent-fill); }
```

Use a real `<table>` with `<thead>`/`<tbody>` (a11y). For sortable/paginated tables in
React, keep semantics and add `aria-sort` on sorted headers.

## Charts (lightweight, dependency-free)

Prefer inline **SVG** for line/area/bar charts — small, crisp, themeable with tokens:

```html
<svg viewBox="0 0 300 100" class="spark" role="img" aria-label="Traffic, last 24h">
  <polyline fill="none" stroke="var(--accent)" stroke-width="2" points="0,80 40,60 ..." />
  <polygon fill="var(--accent-soft)" points="0,100 0,80 40,60 ... 300,100" />
</svg>
```

- **Bars:** CSS flex/grid columns with `height: %` and `background: var(--accent)`.
- **Gauges/rings:** an SVG `<circle>` with `stroke-dasharray`/`stroke-dashoffset`.
- **Heavy/interactive charts:** if the user needs zoom/tooltips/large series, use a
  charting lib (Recharts/visx in React) but pass the CSS tokens as colors. Note the added
  dependency; don't reach for it for simple sparklines.

## Density & states

- Include empty, loading (skeleton shimmer via `--accent-soft`), and error states for
  data regions — real dashboards need them.
- Keep the grid tight but breathable; align numbers right in tables; use the display font
  for labels/metrics and the body font for prose.

## Checklist (dashboards)

- Semantic `<table>`s; nav uses `aria-current="page"`; landmarks (`<nav>`, `<main>`).
- Charts have `role="img"` + `aria-label` (or an accessible summary).
- Responsive: sidebar collapses; 12-col grid stacks under ~900px.
- Loading / empty / error states present. All colors via tokens.
