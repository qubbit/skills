# Target: React Components

Emit `.jsx`/`.tsx` components that share the same accent-token CSS as the HTML modes.
React mode optimizes for **component reuse, correct state, and accessibility** — it does
NOT chase the zero-JS Lighthouse bar (that's the static-HTML landing mode only). Keep the
a11y baseline from `SKILL.md`.

## Token wiring

Put the `:root` block from `references/color-system.md` in a single **`theme.css`** and
import it once at the app root:

```jsx
// main.jsx / layout root
import './theme.css';   // defines --accent and the derived --accent-* / status tokens
```

Components then reference tokens via CSS (module or class), never inline hex:

```css
/* Button.module.css */
.primary { background: var(--accent); color: var(--on-accent); box-shadow: 0 0 24px var(--accent-glow); }
.primary:hover { background: var(--accent-bright); }
.primary:focus-visible { outline: 2px solid var(--accent-ring); outline-offset: 3px; }
```
```jsx
import s from './Button.module.css';
export function Button({ variant = 'primary', ...props }) {
  return <button className={s[variant]} {...props} />;
}
```

**Runtime re-theming:** because everything derives from `--accent`, the whole app
re-themes by setting one CSS variable — no rebuild:

```jsx
<div style={{ '--accent': userColor }}>{/* subtree re-themes */}</div>
// or document.documentElement.style.setProperty('--accent', userColor)
```
Expose this as a prop/context (e.g. `<ThemeProvider accent="#ff3df0">`) when the user
wants selectable themes.

## Styling approach

Match the project's existing convention. In priority order:
1. The repo already uses X (CSS Modules / Tailwind / styled-components / vanilla-extract) → use X, feeding it the tokens.
2. No convention → **CSS Modules** + the shared `theme.css`. Simple, scoped, no deps.
- **Tailwind:** map tokens in `tailwind.config` (`colors.accent: 'var(--accent)'`, etc.) so
  utilities like `bg-accent`/`border-accent-line` resolve to the derived variables.
- Avoid inline `style={{ color: '#00f0ff' }}` — it bypasses the token system.

## State & interactivity

- **Forms:** controlled inputs; validation state per field; render errors below the
  control with `role="alert"` + `aria-invalid`. Mirror the CSS in `references/forms.md`.
  For dropdowns, build the custom listbox from `references/forms.md` (never a native
  `<select>` — its open list can't be themed): keep `open`/`selected`/`activeIndex` in
  `useState`, use the same `role="combobox"`/`role="listbox"` markup and CSS, and show the
  label only on the button (the `✓` lives on the selected option). Or use a headless
  library (Radix/React-Aria `Select`, Headless UI `Listbox`) styled with the tokens — it
  handles the ARIA/keyboard for you.
- **Dashboards/control-centers (live):** fetch/subscribe in `useEffect`; keep a bounded
  buffer; throttle high-frequency updates (rAF or interval) to avoid re-render storms;
  memoize heavy children (`React.memo`, `useMemo`). Announce critical changes via an
  `aria-live` region. See `references/dashboards.md` / `control-center.md`.
- Prefer semantic elements (`<button>`, `<nav>`, `<table>`, `<label>`); add ARIA only to
  fill gaps, not to replace semantics.

## Charts

Small charts → inline SVG components colored with `var(--accent)`/status tokens (no dep).
Heavy/interactive (zoom, tooltips, big series) → a lib (Recharts / visx), passing the CSS
variables as colors. Call out any dependency you add.

## Output

- A focused component per file + the shared `theme.css`.
- If it's a new standalone piece, include a tiny usage snippet.
- If integrating into an existing repo, follow its structure, import style, and TS/JS choice.

## Checklist (React)

- `theme.css` imported once; `--accent` is the single knob; no hardcoded accent hex.
- Components use semantic HTML + focus-visible rings; forms have labeled, announced errors.
- Live UIs are throttled/bounded and announce critical updates; reduced-motion respected.
- Styling matches the repo's convention (or CSS Modules by default).
