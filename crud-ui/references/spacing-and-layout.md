# Spacing & Layout

Whitespace is what separates premium from cheap. Cramped, randomly-spaced UIs read as amateur no matter how good the colors are.

## Spacing Scale

Stick to a 4/8-based scale. Tailwind's defaults already follow it — the discipline is in **reusing the same few values**, not inventing new ones per component.

| Use | Value |
|-----|-------|
| Inline icon gaps, badge padding | `gap-1.5`, `gap-2` (6–8px) |
| Form field internal, tight stacks | `gap-2`, `space-y-2` |
| Between form fields | `space-y-4` / `space-y-5` |
| Card padding | `p-5` / `p-6` |
| Between cards / list items | `gap-4` / `gap-6` |
| Between page sections | `space-y-8` / `space-y-10` |
| Page outer padding | `p-6` mobile → `p-8`/`p-10` desktop |

**Rule of thumb:** when something feels off, it's usually too cramped. Add padding to cards and gap between groups before anything else.

## Border Radius Consistency

Pick a base radius (`--radius: 0.625rem`) and derive the rest. Cards and inputs share it; buttons one step down; badges/avatars full.

- Cards, modals, inputs: `rounded-lg` / `rounded-xl`
- Buttons: `rounded-md` / `rounded-lg`
- Badges, avatars, icon buttons: `rounded-full`

Mixing sharp and round arbitrarily looks unintentional. Be consistent.

## Shadows / Elevation

Modern look = **soft, low shadows + a 1px border**, not heavy drop shadows.

```css
/* subtle, layered — what shadcn cards use */
shadow-sm   /* resting cards */
shadow-md   /* dropdowns, popovers */
shadow-lg   /* modals, command palette */
```

In **dark mode**, shadows barely register — rely on the surface-lift + border instead (see color reference).

## App Shell Layout

The frame that makes it feel like an app. Persistent sidebar + top bar + scrollable content, capped width.

```tsx
export function AppShell({ children }: { children: React.ReactNode }) {
  return (
    <div className="min-h-svh bg-background">
      {/* Sidebar */}
      <aside className="fixed inset-y-0 left-0 hidden w-64 border-r bg-card md:flex md:flex-col">
        <div className="flex h-14 items-center gap-2 border-b px-5">
          <div className="size-7 rounded-md bg-primary" />
          <span className="font-semibold tracking-tight">Acme</span>
        </div>
        <nav className="flex-1 space-y-1 p-3">
          {/* NavItem list */}
        </nav>
        <div className="border-t p-3">{/* user menu */}</div>
      </aside>

      {/* Main column */}
      <div className="md:pl-64">
        {/* Sticky, glassy top bar */}
        <header className="sticky top-0 z-30 flex h-14 items-center gap-3 border-b bg-background/80 px-6 backdrop-blur">
          {/* search, spacer, ThemeToggle, avatar */}
        </header>
        <main className="mx-auto w-full max-w-7xl p-6 lg:p-8">
          {children}
        </main>
      </div>
    </div>
  );
}
```

Nav item with accent active-state:

```tsx
function NavItem({ to, icon: Icon, label }: NavItemProps) {
  return (
    <NavLink
      to={to}
      className={({ isActive }) =>
        cn(
          "flex items-center gap-3 rounded-md px-3 py-2 text-sm font-medium transition-colors",
          isActive
            ? "bg-primary/10 text-primary"
            : "text-muted-foreground hover:bg-accent hover:text-foreground"
        )
      }
    >
      <Icon className="size-4" />
      {label}
    </NavLink>
  );
}
```

Key details: `bg-card` sidebar separated by `border-r`; `h-14` bars; **glassy sticky header** (`bg-background/80 backdrop-blur`) — a current, tasteful touch; content capped at `max-w-7xl` and centered so tables don't stretch across ultrawide monitors.

## Page Layout Pattern

Every list/index page follows the same skeleton — consistency is what makes it feel designed:

```tsx
<div className="space-y-6">
  {/* Header row: title block + primary action */}
  <div className="flex items-start justify-between gap-4">
    <div className="space-y-1">
      <h1 className="text-2xl font-semibold tracking-tight">Patients</h1>
      <p className="text-sm text-muted-foreground">Manage active records.</p>
    </div>
    <Button><Plus className="size-4" /> New patient</Button>
  </div>

  {/* Toolbar: search + filters */}
  <div className="flex items-center gap-2">
    {/* search input, filter dropdowns, density toggle on the right */}
  </div>

  {/* Content card: table / grid / empty / loading */}
  <Card>{/* DataTable or states */}</Card>
</div>
```

## Responsive

- **Sidebar → drawer**: hide the fixed sidebar below `md` (`hidden md:flex`); add a hamburger that opens a `Sheet` (drawer) with the same nav.
- **Tables on mobile**: either horizontal scroll with a frozen first column (`overflow-x-auto`, sticky `left-0` on the name cell) **or** transform rows into stacked cards below `md`. Pick per density.
- **Forms**: single column on mobile, two-column grid (`grid md:grid-cols-2 gap-4`) on desktop for short fields; keep long fields full-width (`md:col-span-2`).
- **Touch targets**: interactive elements ≥ 40px (`h-10`) on touch.
- Use `min-h-svh` (small viewport height) over `min-h-screen` to handle mobile browser chrome.

## Table Density Toggle

For data-heavy tables, let users switch comfortable ↔ compact:

```tsx
const [dense, setDense] = useState(false);
// apply to row cells:
<td className={cn("px-4", dense ? "py-1.5 text-xs" : "py-3 text-sm")}>…</td>
```

Comfortable (`py-3`) is the friendly default; compact (`py-1.5`) for power users scanning hundreds of rows.
