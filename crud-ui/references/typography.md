# Typography & Fonts

Type is half the perceived quality. One well-chosen variable font with a clear hierarchy beats a fancy pairing used carelessly.

## Font Choice

Use **one variable font** for the whole UI. Defaults, in order of preference for CRUD/data apps:

| Font | Feel | Install |
|------|------|---------|
| **Geist** | Modern, neutral, made for product UI (Vercel) | `@fontsource-variable/geist` |
| **Inter Variable** | The workhorse; excellent at small sizes | `@fontsource-variable/inter` |
| **Plus Jakarta Sans** | Friendly, rounder, a bit more personality | `@fontsource-variable/plus-jakarta-sans` |
| **Outfit** | Geometric, distinctive headings | `@fontsource-variable/outfit` |

For a numeric/code accent (IDs, monospace data, code), add **Geist Mono** or **JetBrains Mono**.

**Avoid** the defaults that scream "unstyled": system-ui alone, Arial, Roboto, Times.

### Loading (no layout shift, no network)

```ts
// main.tsx — variable font, one import, all weights
import "@fontsource-variable/geist";
```

```css
/* index.css */
@theme inline {
  --font-sans: "Geist Variable", ui-sans-serif, system-ui, sans-serif;
  --font-mono: "Geist Mono Variable", ui-monospace, monospace;
}
body { font-family: var(--font-sans); }
```

Prefer `@fontsource-variable/*` over Google Fonts `<link>` — self-hosted = no FOUT, no third-party request, no CLS.

## Type Scale

Don't pick sizes ad hoc. Use this scale and assign roles:

| Token | Size | Role |
|-------|------|------|
| `text-xs` | 12px | Table meta, captions, uppercase labels, badges |
| `text-sm` | 14px | **Body default in dense UIs**, table cells, form inputs |
| `text-base` | 16px | Body in roomier layouts, paragraph text |
| `text-lg` | 18px | Card titles, section subheads |
| `text-xl` | 20px | Section titles |
| `text-2xl` | 24px | Page titles |
| `text-3xl`+ | 30px+ | Dashboard hero numbers, marketing |

CRUD apps generally run on `text-sm` as the body size (denser, more pro), with `text-xs` for metadata.

## Hierarchy Through Weight & Color, Not Just Size

This is the core CRUD typography move. A label/value pair:

```tsx
<div>
  <dt className="text-xs font-medium uppercase tracking-wide text-muted-foreground">
    Status
  </dt>
  <dd className="text-sm font-medium text-foreground">Active</dd>
</div>
```

- **Muted color** (`text-muted-foreground`) for labels, captions, secondary info.
- **Foreground + medium weight** for the actual data.
- Three weights are plenty: `font-normal` (body), `font-medium` (labels, emphasis, buttons), `font-semibold` (titles). Avoid `font-bold` everywhere — it's heavy-handed.

## Letter-Spacing

- **Large headings**: `tracking-tight` (-0.02em) — tightens the gappy look of big text.
- **Body**: leave default.
- **Tiny uppercase labels**: `uppercase tracking-wide text-xs` — uppercase needs the extra spacing to be readable.

## Tabular Numbers (critical for data tables)

Default proportional figures make columns of numbers misalign. Enable tabular figures so digits share a width:

```tsx
<td className="text-sm tabular-nums">{formatCurrency(amount)}</td>
```

```css
/* or globally for all table/number cells */
.tabular { font-variant-numeric: tabular-nums; }
```

Apply to: currency, quantities, dates, IDs, anything in a column you scan vertically. Pair with `text-right` for numeric columns.

## Line Height & Measure

- Body text: `leading-relaxed` (1.625) for paragraphs; default `leading-normal` for UI.
- Constrain paragraph width to `max-w-prose` (~65ch) — full-width body text is hard to read.
- Headings: `leading-tight`.

## Quick Hierarchy Recipe for a Page Header

```tsx
<header className="space-y-1">
  <h1 className="text-2xl font-semibold tracking-tight">Patients</h1>
  <p className="text-sm text-muted-foreground">
    Manage active and archived patient records.
  </p>
</header>
```

That two-line header (title + muted subtitle) instantly reads as "designed."
