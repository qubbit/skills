# Configurable Color System — One Accent, Derived Variants

Every space-ui artifact is themed by **one** author-chosen accent color. All other
accent variants (hover, glow, border, muted, tint, ring) are **derived from it with
CSS color math** — never hand-picked. Change one value, the whole UI re-themes.

This is the single source of truth for color. Every mode (landing, forms,
dashboards, control-center) imports this token block. Do not hardcode hex accents
anywhere else.

## The rule

1. Author sets exactly one accent: `--accent`. (Default cyan `#00f0ff` if unspecified.)
2. Everything else is computed from `--accent` using **`oklch(from …)` relative color
   syntax** and `color-mix()`. Use OKLCH, not HSL: OKLCH lightness/chroma steps are
   perceptually uniform (equal steps look equal across hues), and `calc()` on its
   `<number>` channels (`l`, `c`) works reliably — whereas `calc(l + 12%)` inside
   `hsl(from …)` mixes number+percentage and silently fails to transparent in current
   engines. `color-mix()` handles the translucent variants.
3. The neutral/base scale (backgrounds, ink, lines) is fixed dark-mode-first and does
   NOT change with the accent — only the accent family re-themes.

## Token block (drop this in `:root`)

```css
:root {
  /* ---- The ONE knob. Change this to re-theme everything. ---- */
  --accent: #00f0ff;

  /* ---- Derived accent family (OKLCH color math — do not hand-pick) ---- */
  /* oklch(from C  L  C  H) — reorder is fine; keep channel roles. l,c are <number>. */
  /* Brighter accent for hover (raise lightness). */
  --accent-bright: oklch(from var(--accent) calc(l + 0.08) c h);
  /* Deeper accent for pressed/active (drop lightness). */
  --accent-deep:   oklch(from var(--accent) calc(l - 0.10) c h);
  /* Muted accent for secondary text/icons (dim + reduce chroma). */
  --accent-muted:  oklch(from var(--accent) calc(l - 0.06) calc(c * 0.7) h);

  /* Translucent accents for glows, fills, borders (color-mix keeps alpha math simple). */
  --accent-soft:   color-mix(in srgb, var(--accent) 12%, transparent);
  --accent-fill:   color-mix(in srgb, var(--accent) 6%,  transparent);
  --accent-line:   color-mix(in srgb, var(--accent) 22%, transparent);
  --accent-ring:   color-mix(in srgb, var(--accent) 45%, transparent);
  --accent-glow:   color-mix(in srgb, var(--accent) 30%, transparent);

  /* On-accent text: near-black tinted toward the accent so filled buttons read well. */
  --on-accent: color-mix(in srgb, var(--accent) 12%, #04050a);

  /* ---- Fixed neutral base (independent of accent) ---- */
  --bg:    #06070d;
  --bg-2:  #080a14;
  --panel: #0a0c16;
  --ink:   #e6ebff;   /* primary text */
  --muted: #a8b1d6;   /* secondary text */
  --dim:   #7f89b0;   /* labels / tertiary */
  --line:  color-mix(in srgb, var(--accent) 14%, transparent); /* accent-tinted hairline */

  /* ---- Semantic status colors (for dashboards/control-centers) ---- */
  /* Derived from the accent's lightness/chroma but re-hued to the conventional
     status hues, so they read as status yet stay in the theme's "family".
     Bump chroma a touch and cap lightness so red/amber/green are legible. */
  --ok:    oklch(from var(--accent) clamp(0.55, l, 0.82) max(c, 0.16) 150); /* green  */
  --warn:  oklch(from var(--accent) clamp(0.55, l, 0.85) max(c, 0.16) 85);  /* amber  */
  --crit:  oklch(from var(--accent) clamp(0.55, l, 0.70) max(c, 0.18) 25);  /* red    */
  --info:  var(--accent);
}
```

## Fallback for older browsers

Relative color syntax is Chrome 119+ / Safari 16.4+ / Firefox 128+ (all 2024+).
`color-mix()` is slightly broader (2023+). For a static landing page that must hit
Lighthouse on old engines, provide static fallbacks BEFORE the computed values so
unsupported browsers use the literal and supported ones override:

```css
:root {
  --accent: #00f0ff;
  --accent-bright: #5cf6ff;          /* static fallback */
  --accent-bright: hsl(from var(--accent) h s calc(l + 12%)); /* override where supported */
}
```

Only bother with fallbacks in the **static-HTML landing mode** (Lighthouse-critical).
Forms / dashboards / control-centers are app UIs on modern engines — use the computed
values directly. If the user names a target browser, honor it.

## Usage patterns

- **Luminous border:** `border: 1px solid var(--accent-line);`
- **Hover glow:** `box-shadow: 0 0 24px var(--accent-glow);`
- **Focus ring:** `outline: 2px solid var(--accent-ring); outline-offset: 2px;`
- **Filled primary button:** `background: var(--accent); color: var(--on-accent);`
  hover → `background: var(--accent-bright);`
- **Ghost/secondary button:** `color: var(--ink); border: 1px solid var(--line);`
  hover → `border-color: var(--accent-ring); color: var(--accent);`
- **Panel fill:** `background: var(--accent-fill);`
- **Soft chip/badge:** `background: var(--accent-soft); color: var(--accent);`

## Multiple accents (advanced)

If a UI needs a second accent (e.g. a control-center with two subsystems), define a
second knob `--accent-2` and derive its family with the SAME formulas. Never exceed
two accents plus status colors — restraint is the aesthetic.

## Picking a good accent

Any vivid, high-lightness hue works on the dark base. Verify WCAG contrast of
`--accent` as text on `--bg` (aim ≥ 4.5:1 for body, ≥ 3:1 for large/UI). If an accent
is too dark to read as text, use it only for fills/borders and use `--ink` for text.
Suggested starting points: cyan `#00f0ff`, magenta `#ff3df0`, amber `#ffb020`,
violet `#8b5cff`, lime `#9cff57`, ice-blue `#5cc8ff`.
