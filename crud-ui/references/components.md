# Components & Patterns

CRUD-specific building blocks. All assume shadcn/ui primitives (`npx shadcn@latest add button input table dialog dropdown-menu badge card`) and the token system from `color-and-theme.md`. Adapt class names if not using shadcn, but keep the structure and state coverage.

## Buttons — every variant, every state

shadcn's button already gives you variants via `cva`. The non-negotiables:

```tsx
// Primary action
<Button>Save changes</Button>
// Secondary
<Button variant="outline">Cancel</Button>
// Low-emphasis (toolbars, table rows)
<Button variant="ghost" size="sm">Edit</Button>
// Destructive
<Button variant="destructive">Delete</Button>

// Loading state — disable + spinner, keep width stable
<Button disabled={isPending}>
  {isPending && <Loader2 className="size-4 animate-spin" />}
  {isPending ? "Saving…" : "Save"}
</Button>
```

Every button must have, via the base classes: `transition-colors`, a `hover:` shade, `focus-visible:ring-2 focus-visible:ring-ring`, and `disabled:opacity-50 disabled:pointer-events-none`. shadcn bakes these in — don't strip them.

## Status Badge — color + shape, not just text

```tsx
const statusStyles = {
  active:   "bg-emerald-500/10 text-emerald-600 dark:text-emerald-400 ring-emerald-500/20",
  pending:  "bg-amber-500/10 text-amber-600 dark:text-amber-400 ring-amber-500/20",
  archived: "bg-zinc-500/10 text-zinc-600 dark:text-zinc-400 ring-zinc-500/20",
  error:    "bg-red-500/10 text-red-600 dark:text-red-400 ring-red-500/20",
} as const;

export function StatusBadge({ status }: { status: keyof typeof statusStyles }) {
  return (
    <span className={cn(
      "inline-flex items-center gap-1.5 rounded-full px-2 py-0.5 text-xs font-medium ring-1 ring-inset",
      statusStyles[status]
    )}>
      <span className="size-1.5 rounded-full bg-current" />
      {status}
    </span>
  );
}
```

The dot + tinted background + ring is the current pattern. Avoid solid-fill, full-saturation badges.

## Data Table

Use `@tanstack/react-table` for logic, render with shadcn `Table`. Essential features: sortable headers, row hover, sticky header, per-row action menu, pagination, tabular numbers on numeric columns.

```tsx
<div className="overflow-hidden rounded-xl border bg-card">
  <table className="w-full text-sm">
    <thead className="sticky top-0 bg-muted/50 backdrop-blur">
      <tr className="border-b">
        <th className="px-4 py-2.5 text-left font-medium text-muted-foreground">
          <button className="inline-flex items-center gap-1 hover:text-foreground">
            Name <ChevronsUpDown className="size-3.5" />
          </button>
        </th>
        <th className="px-4 py-2.5 text-right font-medium text-muted-foreground">Amount</th>
        <th className="w-10" />
      </tr>
    </thead>
    <tbody>
      {rows.map((row) => (
        <tr key={row.id} className="border-b transition-colors last:border-0 hover:bg-muted/40">
          <td className="px-4 py-3 font-medium">{row.name}</td>
          <td className="px-4 py-3 text-right tabular-nums">{formatCurrency(row.amount)}</td>
          <td className="px-2 py-3">
            <DropdownMenu>
              <DropdownMenuTrigger asChild>
                <Button variant="ghost" size="icon" className="size-8">
                  <MoreHorizontal className="size-4" />
                </Button>
              </DropdownMenuTrigger>
              <DropdownMenuContent align="end">
                <DropdownMenuItem onSelect={() => onEdit(row)}>
                  <Pencil className="size-4" /> Edit
                </DropdownMenuItem>
                <DropdownMenuSeparator />
                <DropdownMenuItem
                  className="text-destructive focus:text-destructive"
                  onSelect={() => onDelete(row)}
                >
                  <Trash2 className="size-4" /> Delete
                </DropdownMenuItem>
              </DropdownMenuContent>
            </DropdownMenu>
          </td>
        </tr>
      ))}
    </tbody>
  </table>
</div>
```

Notes: header is `text-muted-foreground` + `font-medium` (not bold); rows separate with `border-b` not full grids; hover is a soft `bg-muted/40`; numeric columns are `text-right tabular-nums`; the delete item is `text-destructive`. Wrap in `overflow-x-auto` for responsive scroll.

Pagination row below the table:

```tsx
<div className="flex items-center justify-between px-4 py-3 text-sm text-muted-foreground">
  <span>{selected} of {total} selected</span>
  <div className="flex items-center gap-2">
    <Button variant="outline" size="sm" disabled={!canPrev}>Previous</Button>
    <Button variant="outline" size="sm" disabled={!canNext}>Next</Button>
  </div>
</div>
```

## Forms (react-hook-form + zod)

```tsx
const schema = z.object({
  name: z.string().min(1, "Name is required"),
  email: z.string().email("Enter a valid email"),
  role: z.enum(["admin", "member"]),
});

function PatientForm({ onSubmit }: Props) {
  const form = useForm<z.infer<typeof schema>>({ resolver: zodResolver(schema) });
  const { isSubmitting } = form.formState;

  return (
    <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
      {/* Grouped section */}
      <div className="space-y-1">
        <h2 className="text-sm font-semibold">Details</h2>
        <p className="text-sm text-muted-foreground">Basic patient information.</p>
      </div>

      <div className="grid gap-4 md:grid-cols-2">
        <Field label="Full name" error={form.formState.errors.name?.message}>
          <Input {...form.register("name")} placeholder="Jane Doe" />
        </Field>
        <Field label="Email" error={form.formState.errors.email?.message}>
          <Input type="email" {...form.register("email")} placeholder="jane@…" />
        </Field>
      </div>

      {/* Sticky save bar */}
      <div className="sticky bottom-0 -mx-6 flex justify-end gap-2 border-t bg-background/80 px-6 py-3 backdrop-blur">
        <Button type="button" variant="outline">Cancel</Button>
        <Button type="submit" disabled={isSubmitting}>
          {isSubmitting && <Loader2 className="size-4 animate-spin" />}
          Save patient
        </Button>
      </div>
    </form>
  );
}
```

Reusable field wrapper — label + helper + inline error:

```tsx
function Field({ label, error, hint, children }: FieldProps) {
  return (
    <div className="space-y-1.5">
      <label className="text-sm font-medium">{label}</label>
      {children}
      {hint && !error && <p className="text-xs text-muted-foreground">{hint}</p>}
      {error && <p className="text-xs font-medium text-destructive">{error}</p>}
    </div>
  );
}
```

When a field errors, also add `aria-invalid` so the input shows a red ring:
```css
/* shadcn Input already handles: */
aria-invalid:ring-destructive/20 aria-invalid:border-destructive
```

Form principles: group related fields with a section header + muted description; two-column grid for short fields, full-width for long ones; helper text in muted, errors in destructive; **sticky save bar** so the primary action is always reachable; disable submit + show spinner while pending.

## Detail / Read View — definition list

```tsx
<dl className="grid gap-x-6 gap-y-4 sm:grid-cols-2">
  <Detail label="Status"><StatusBadge status="active" /></Detail>
  <Detail label="Email" value="jane@example.com" />
  <Detail label="Created" value="Jun 9, 2026" />
</dl>

function Detail({ label, value, children }: DetailProps) {
  return (
    <div className="space-y-1">
      <dt className="text-xs font-medium uppercase tracking-wide text-muted-foreground">{label}</dt>
      <dd className="text-sm font-medium text-foreground">{children ?? value}</dd>
    </div>
  );
}
```

## Card

```tsx
<div className="rounded-xl border bg-card p-6 shadow-sm">
  <div className="space-y-1">
    <h3 className="font-semibold tracking-tight">Section title</h3>
    <p className="text-sm text-muted-foreground">Supporting description.</p>
  </div>
  <div className="mt-5">{/* content */}</div>
</div>
```

## The `cn` Helper

All of the above rely on it (shadcn ships this):

```ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";
export function cn(...inputs: ClassValue[]) { return twMerge(clsx(inputs)); }
```
