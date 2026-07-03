# Color & Dark Mode

The single highest-leverage area. Get the token system right and every component inherits good color for free.

## Core Rules

1. **Never pure black or white.** `#fff` and `#000` are harsh and amateur.
   - Light background: `#fafafa`, `#f8fafc`, or `#fcfcfc`.
   - Dark background: `#0a0a0a`–`#101012` (a near-black with a hint of the neutral's temperature).
   - Text: near-black (`zinc-900` ≈ `#18181b`) on light, near-white (`zinc-100` ≈ `#f4f4f5`) on dark.
2. **Layer surfaces** so depth reads without heavy shadows. Each surface a measurable step apart:
   `background` → `card` → `popover/elevated` → `border`.
3. **One accent**, used only for: primary actions, active nav, focus rings, links, selected states. Everything else neutral.
4. **Pick a neutral family on purpose** — they have temperature:
   - `zinc` / `neutral` — true neutral, modern, safe default.
   - `slate` — cool, bluish, "techy / SaaS".
   - `stone` — warm, "editorial / organic".
   - `gray` — slightly cool, classic.
5. **Semantic colors** for state only: success `emerald`, warning `amber`, danger `red`/`rose`, info `blue`/`sky`. Use **tinted** backgrounds for badges (`bg-emerald-50 text-emerald-700` / dark `bg-emerald-500/10 text-emerald-400`), not full saturation.

## Accent / Neutral Pairings That Look Current

Pick one row. All avoid the default `blue-500`-for-everything look.

| Vibe | Neutral | Accent |
|------|---------|--------|
| Modern SaaS (safe default) | `zinc` | `violet` / `indigo` |
| Fintech / trustworthy | `slate` | `blue` (deeper, e.g. `blue-600`) / `sky` |
| Fresh / health / growth | `zinc` | `emerald` / `teal` |
| Bold / creative | `neutral` | `fuchsia` / `rose` |
| Premium / editorial | `stone` | `amber` / `orange` |
| Developer tool | `zinc` | `cyan` / `lime` (sparingly) |

## Token System (Tailwind v4 + shadcn convention)

Tailwind v4 is **CSS-first** — no `tailwind.config.js`. Define tokens as CSS variables and map them in `@theme inline`. This is the shadcn/ui v4 convention. Components then use `bg-background`, `text-muted-foreground`, `border-border`, etc., and dark mode "just works" by swapping variables under `.dark`.

```css
/* src/index.css */
@import "tailwindcss";
@import "tw-animate-css";

@custom-variant dark (&:is(.dark *));

:root {
  --radius: 0.625rem;

  /* Light theme — OKLCH for perceptual consistency */
  --background: oklch(0.99 0 0);            /* near-white, not #fff */
  --foreground: oklch(0.21 0.01 285);       /* near-black */
  --card: oklch(1 0 0);
  --card-foreground: oklch(0.21 0.01 285);
  --popover: oklch(1 0 0);
  --popover-foreground: oklch(0.21 0.01 285);

  --primary: oklch(0.55 0.22 285);          /* violet accent */
  --primary-foreground: oklch(0.98 0 0);
  --secondary: oklch(0.97 0 0);
  --secondary-foreground: oklch(0.21 0.01 285);
  --muted: oklch(0.97 0 0);
  --muted-foreground: oklch(0.55 0.01 285); /* secondary text */
  --accent: oklch(0.97 0 0);
  --accent-foreground: oklch(0.21 0.01 285);

  --destructive: oklch(0.58 0.22 27);       /* red */
  --success: oklch(0.6 0.16 150);           /* emerald */
  --warning: oklch(0.75 0.16 75);           /* amber */

  --border: oklch(0.92 0.004 285);
  --input: oklch(0.92 0.004 285);
  --ring: oklch(0.55 0.22 285);             /* = primary, for focus rings */
}

.dark {
  --background: oklch(0.16 0.004 285);      /* near-black, not #000 */
  --foreground: oklch(0.96 0 0);
  --card: oklch(0.19 0.004 285);            /* lifted off background */
  --card-foreground: oklch(0.96 0 0);
  --popover: oklch(0.21 0.004 285);
  --popover-foreground: oklch(0.96 0 0);

  --primary: oklch(0.65 0.2 285);           /* slightly brighter in dark */
  --primary-foreground: oklch(0.16 0.004 285);
  --secondary: oklch(0.25 0.004 285);
  --secondary-foreground: oklch(0.96 0 0);
  --muted: oklch(0.25 0.004 285);
  --muted-foreground: oklch(0.68 0.01 285);
  --accent: oklch(0.27 0.004 285);
  --accent-foreground: oklch(0.96 0 0);

  --destructive: oklch(0.65 0.19 25);
  --success: oklch(0.68 0.15 150);
  --warning: oklch(0.78 0.15 75);

  --border: oklch(1 0 0 / 10%);             /* borders MORE visible in dark */
  --input: oklch(1 0 0 / 14%);
  --ring: oklch(0.65 0.2 285);
}

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-card: var(--card);
  --color-card-foreground: var(--card-foreground);
  --color-popover: var(--popover);
  --color-popover-foreground: var(--popover-foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-secondary: var(--secondary);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);
  --color-accent: var(--accent);
  --color-accent-foreground: var(--accent-foreground);
  --color-destructive: var(--destructive);
  --color-success: var(--success);
  --color-warning: var(--warning);
  --color-border: var(--border);
  --color-input: var(--input);
  --color-ring: var(--ring);
  --radius-sm: calc(var(--radius) - 4px);
  --radius-md: calc(var(--radius) - 2px);
  --radius-lg: var(--radius);
  --radius-xl: calc(var(--radius) + 4px);
}

* { border-color: var(--color-border); }
body { background: var(--color-background); color: var(--color-foreground); }
```

> **Why OKLCH?** Perceptually uniform — lightness changes look even, and you can tweak just the lightness channel to derive surface layers. If a project already uses hex/HSL, match it; don't churn it for purity. To re-accent, change the hue (third value) of `--primary` and `--ring` together.

## Dark Mode Toggle (next-themes)

```tsx
// main.tsx
import { ThemeProvider } from "next-themes";
<ThemeProvider attribute="class" defaultTheme="system" enableSystem>
  <App />
</ThemeProvider>
```

```tsx
// ThemeToggle.tsx
import { useTheme } from "next-themes";
import { Moon, Sun } from "lucide-react";
import { Button } from "@/components/ui/button";

export function ThemeToggle() {
  const { setTheme, resolvedTheme } = useTheme();
  return (
    <Button
      variant="ghost" size="icon"
      onClick={() => setTheme(resolvedTheme === "dark" ? "light" : "dark")}
      aria-label="Toggle theme"
    >
      <Sun className="size-4 rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
      <Moon className="absolute size-4 rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
    </Button>
  );
}
```

Add `suppressHydrationWarning` on `<html>` (SSR) to avoid theme-flash warnings.

## Dark Mode Design Adjustments (not just inversion)

- **Lift cards off the background** (`--card` lighter than `--background`) instead of relying on shadows — shadows barely read on dark.
- **Make borders more visible** (`white / 10–14%`) — they do the separation work shadows did in light mode.
- **Slightly brighten the accent** so it stays vivid against dark.
- **Dial back pure-white text** to `~96%` lightness to avoid glare.
- **Tinted semantic badges**: `bg-emerald-500/10 text-emerald-400` reads far better than `bg-emerald-50` in dark.

## Common Mistakes

- Hard-coding `bg-white`/`text-black`/`bg-gray-100` in components → breaks dark mode. Always use tokens (`bg-card`, `text-foreground`, `bg-muted`).
- Using `blue-500` as both accent and info → no distinction. Reserve blue for info; pick a different accent.
- Full-saturation badge backgrounds → loud and dated. Tint them.
- Same shadow in light and dark → invisible in dark. Reduce shadow reliance; lean on borders + surface lift.
