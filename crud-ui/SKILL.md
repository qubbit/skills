---
name: crud-ui
description: Build modern, polished UIs for CRUD apps — create new apps or modernize existing ones. Use whenever the user wants to make a data app (dashboards, admin panels, tables, forms, list/detail views, internal tools, SaaS-style apps) look good, less boring, or more "trendy" / professional. Triggers on "make this UI nicer", "design a CRUD app", "build an admin dashboard", "this looks boring/dated", "add dark mode", "improve the forms/tables", "deletion confirmation", "loading and error states", or any request to style a React + Tailwind data interface. Covers dark mode, color systems, typography/fonts, spacing, deletion confirmation, error/loading/empty states, and contemporary visual patterns.
---

# CRUD UI — Modern Interfaces for Data Apps

Build CRUD interfaces that feel like a 2025-era SaaS product, not a Bootstrap admin template from 2016. The goal: clean, confident, slightly opinionated UIs with real depth (dark mode, motion, thoughtful states) — never boring, never generic.

This skill works two ways:
1. **New app** — scaffold a polished CRUD app from scratch.
2. **Existing app** — audit and modernize what's already there without a rewrite.

## Default Stack

Unless the project already dictates otherwise, target:

- **React + TypeScript + Vite**
- **Tailwind CSS v4** (CSS-first config via `@theme`, not `tailwind.config.js`)
- **shadcn/ui** for primitives (Radix-based, you own the code, fully restyleable)
- **lucide-react** for icons
- **sonner** for toasts
- **@tanstack/react-table** for data tables
- A variable font (Geist, Inter Variable, or Plus Jakarta Sans)

If the existing project uses something else (Vue, plain CSS, MUI, Chakra), adapt the *principles* below — they're framework-agnostic. Don't force a migration unless asked.

## How to Work

### When modernizing an existing app

1. **Look before you touch.** Read the entry CSS, the theme/Tailwind config, 2-3 representative components (a table, a form, a layout shell), and how it handles loading/errors today. Use `Glob`/`Grep` to find them.
2. **Diagnose the boring.** Run the "Why it looks boring" checklist below and name the specific offenders (flat grays, no dark mode, default font, cramped spacing, no empty states, browser `confirm()` for deletes).
3. **Propose, then apply.** Lead with the 3-5 highest-leverage changes. Establish the design tokens *first* (color + font + spacing scale), because every component change depends on them.
4. **Preserve behavior.** Modernizing visuals must not break data flow, routing, or form logic. Restyle; don't rearchitect unless asked.

### When building new

1. Scaffold tokens → layout shell → one entity end-to-end (list → create → edit → delete) → then replicate the pattern.
2. Get one entity *fully* polished (all states: loading, empty, error, success) before scaling out. A consistent pattern beats six half-done screens.

## Why It Looks Boring (the diagnosis checklist)

Most dull CRUD UIs fail in predictable ways. Check for these:

- [ ] **One flat gray everywhere** — no layering. Surfaces don't separate from the background.
- [ ] **No dark mode** — or a dark mode that's just inverted with muddy contrast.
- [ ] **Default system font** (or Arial/Roboto) at one weight.
- [ ] **Cramped or random spacing** — no consistent rhythm; everything 8px apart.
- [ ] **Pure black text on pure white** (`#000` on `#fff`) — harsh, amateur.
- [ ] **Saturated, default Tailwind `blue-500` as the only accent**, used for everything.
- [ ] **No empty / loading / error states** — blank screens or raw spinners.
- [ ] **Browser `alert()` / `confirm()`** for feedback and deletion.
- [ ] **Tables that are just `<table>` with borders** — no hierarchy, no hover, no density control.
- [ ] **No motion** — nothing eases in; state changes snap.
- [ ] **Buttons with no states** — no hover, focus ring, disabled, or loading.

Fixing even the first four transforms the feel immediately.

## The Five Pillars

Read these in order. The deep material for each lives in `references/`.

### 1. Color & Dark Mode → `references/color-and-theme.md`

The single biggest lever. Rules:

- **Never pure black or pure white.** Light bg `#fafafa`/`#f8fafc`, not `#fff`. Dark bg `#0a0a0a`–`#101012`, not `#000`. Text is near-black/near-white (e.g. `zinc-900` / `zinc-100`), not absolute.
- **Layer your surfaces.** Background → card → elevated (popover/modal) → border. Each step a measurable shade apart, so depth reads without heavy shadows.
- **One accent color, used deliberately** — primary actions, active nav, focus rings, links. Everything else is neutral. Pick a neutral *family* (zinc/slate/stone/neutral — they have different temperatures) and ONE accent (violet, indigo, emerald, blue, rose...). Avoid raw `blue-500`.
- **Semantic colors** for state: success (emerald), warning (amber), danger (red/rose), info (blue). Use desaturated/tinted backgrounds for badges, not full saturation.
- **Dark mode is a first-class design, not an inversion.** Reduce contrast slightly, lower shadow reliance, raise border visibility. Define both themes as CSS variables and let components reference tokens (`bg-background`, `text-muted-foreground`) — never hard-coded hex in components.

Implement with Tailwind v4 `@theme` + CSS variables + a `.dark` class toggled by `next-themes`. See the reference for a complete copy-paste token block.

### 2. Typography & Fonts → `references/typography.md`

- **One good variable font** beats two mediocre ones. Defaults: **Geist** (modern, neutral), **Inter Variable** (workhorse), **Plus Jakarta Sans** (friendlier), **Outfit** (geometric). For numeric/data-heavy tables, enable **tabular figures** (`font-variant-numeric: tabular-nums`).
- **Type scale**, don't pick sizes at random: `text-xs` (labels/meta) → `sm` (body/table) → `base` → `lg`/`xl` (section titles) → `2xl`+ (page titles). Establish hierarchy with **weight and color**, not just size — a `font-medium text-foreground` label over `text-sm text-muted-foreground` value is the core CRUD pattern.
- **Letter-spacing**: tighten large headings slightly (`tracking-tight`), leave body alone, widen tiny uppercase labels (`tracking-wide` + `uppercase` + `text-xs`).
- Load fonts via `@fontsource-variable/*` (no network, no layout shift) rather than Google Fonts `<link>`.

### 3. Spacing & Layout → `references/spacing-and-layout.md`

- **Rhythm via a scale**: stick to 4/8-based spacing (`4, 6, 8, 12, 16, 24...`). Pick consistent values for card padding (e.g. `p-6`), gaps between cards (`gap-4`/`gap-6`), and section spacing (`space-y-8`).
- **Generous whitespace** is what makes premium UIs feel premium. Cramped = cheap. When in doubt, add more padding to cards and more gap between groups.
- **Max content width** (`max-w-7xl mx-auto`) + a persistent sidebar or top nav. Don't let tables stretch edge-to-edge on wide monitors.
- **Density control** for tables (comfortable vs compact rows) when data-heavy.
- **Responsive**: sidebar collapses to a sheet/drawer on mobile; tables become cards or scroll horizontally with a frozen first column.

### 4. Components & Patterns → `references/components.md`

The CRUD-specific building blocks, with code:

- **App shell**: sidebar nav (active state via accent), top bar with search + theme toggle + user menu.
- **Data table**: sortable headers, row hover, row selection, pagination, density toggle, column visibility, sticky header, per-row action menu (`⋯`). Tabular numbers for numeric columns.
- **Forms**: labeled fields with helper text, inline validation (zod + react-hook-form), grouped sections, sticky save bar, disabled-while-submitting buttons with spinners.
- **Detail / list views**: definition-list pattern (label muted, value emphasized), status badges, metadata rows.
- **Buttons**: primary/secondary/ghost/destructive variants, every one with hover + focus-visible ring + disabled + loading states.

### 5. States: Loading, Empty, Error, Delete → `references/states-and-feedback.md`

This is what separates a real product from a demo. Every async surface needs all four.

- **Loading**: **skeleton screens** that match the final layout (not centered spinners). Shimmer animation. For buttons, inline spinner + disabled.
- **Empty**: a centered illustration/icon, a one-line explanation, and a primary CTA ("Create your first invoice"). Never a blank screen.
- **Error**: an inline error card with a clear message and a **Retry** button. For form fields, red text + ring under the offending field. For mutations, an error **toast** (sonner) — never a silent failure or `alert()`.
- **Success**: a toast confirmation for mutations ("Patient archived"), optimistic UI where safe, with rollback on failure.
- **Deletion confirmation** — never `window.confirm()`. Use an `AlertDialog`:
  - Title states the consequence ("Delete this aide?").
  - Body names the specific item and whether it's reversible.
  - Destructive action is a **red** button; cancel is the default/safe choice.
  - For irreversible/bulk deletes, require typing the item name to confirm.
  - Prefer **soft delete + Undo toast** over hard delete where the domain allows it — it's both safer and a nicer UX.

## What "Trendy, Not Boring" Means in 2025

Apply these tastefully — restraint reads as premium; piling them all on reads as a template demo:

- **Subtle depth**: low-spread, soft shadows + 1px borders that are slightly lighter/darker than the surface. Layered surfaces over heavy drop shadows.
- **Rounded, not pill-everything**: `rounded-lg`/`rounded-xl` on cards; reserve full `rounded-full` for avatars, badges, icon buttons.
- **Micro-interactions**: 150–200ms ease transitions on hover/color/transform. Buttons nudge, rows highlight, dialogs scale-fade in. Use `tw-animate-css` or Radix's data-state animations.
- **Accent gradients, used sparingly**: a single hero stat card or the active nav indicator — not every button.
- **Glassmorphism in moderation**: a backdrop-blurred sticky top bar (`backdrop-blur bg-background/80`) is current and tasteful. Don't blur everything.
- **Consistent iconography** (lucide) at one stroke width, sized to the text.
- **Status as color + shape**: dot + label badges, not just colored text.
- **Keyboard & focus polish**: visible `focus-visible` rings in the accent color; `⌘K` command palette is a strong "this is a real product" signal if scope allows.

## Quality Bar (self-check before declaring done)

- [ ] Light AND dark mode both look intentional; no hard-coded hex in components — all via tokens.
- [ ] No pure `#000`/`#fff`. Text contrast passes WCAG AA (4.5:1 body).
- [ ] One accent color, consistent neutral family. Semantic colors only for state.
- [ ] One variable font, clear type hierarchy, tabular nums in data tables.
- [ ] Consistent spacing scale; cards have breathing room.
- [ ] Every async view has loading (skeleton), empty, and error states.
- [ ] Mutations give toast feedback; failures surface, never silent.
- [ ] Deletion uses an AlertDialog (or soft-delete + Undo), never `confirm()`.
- [ ] All interactive elements have hover + focus-visible + disabled states.
- [ ] Motion is present but subtle (150–200ms), nothing janky or excessive.
- [ ] Responsive: works at mobile, tablet, desktop widths.

## Reference Files

- `references/color-and-theme.md` — token system, dark mode, copy-paste CSS variables, Tailwind v4 `@theme`, accent/neutral pairings.
- `references/typography.md` — font choices, type scale, weight/color hierarchy, tabular numbers, loading fonts.
- `references/spacing-and-layout.md` — spacing scale, app shell layout, responsive patterns, table density.
- `references/components.md` — copy-paste shadcn-based table, form, button, app-shell, badge code.
- `references/states-and-feedback.md` — skeletons, empty states, error cards, deletion dialogs, toasts, optimistic updates.
