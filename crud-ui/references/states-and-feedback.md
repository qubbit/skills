# States: Loading, Empty, Error, Delete & Feedback

This is what separates a real product from a demo. **Every async surface needs four states**: loading, empty, error, and success. Skipping them is the most common reason a CRUD app feels unfinished.

A clean pattern to enforce it — render by state, never assume data exists:

```tsx
if (query.isPending) return <TableSkeleton rows={8} />;
if (query.isError)   return <ErrorState onRetry={query.refetch} />;
if (!query.data.length) return <EmptyState onCreate={openCreate} />;
return <DataTable rows={query.data} />;
```

## 1. Loading — Skeletons, not spinners

Skeletons that mirror the final layout feel ~2× faster than a centered spinner because the user sees structure immediately. Reserve spinners for buttons/inline.

```tsx
function Skeleton({ className }: { className?: string }) {
  return <div className={cn("animate-pulse rounded-md bg-muted", className)} />;
}

function TableSkeleton({ rows = 6 }: { rows?: number }) {
  return (
    <div className="overflow-hidden rounded-xl border bg-card">
      <div className="border-b bg-muted/50 px-4 py-2.5">
        <Skeleton className="h-4 w-24" />
      </div>
      {Array.from({ length: rows }).map((_, i) => (
        <div key={i} className="flex items-center gap-4 border-b px-4 py-3.5 last:border-0">
          <Skeleton className="size-8 rounded-full" />
          <Skeleton className="h-4 w-40" />
          <Skeleton className="ml-auto h-4 w-16" />
        </div>
      ))}
    </div>
  );
}
```

For a richer shimmer, sweep a gradient instead of a flat pulse:
```css
.shimmer {
  background: linear-gradient(90deg, var(--muted) 25%, color-mix(in oklch, var(--muted), white 8%) 37%, var(--muted) 63%);
  background-size: 400% 100%;
  animation: shimmer 1.4s ease infinite;
}
@keyframes shimmer { 0% { background-position: 100% 0 } 100% { background-position: 0 0 } }
```

**Button loading**: disable + inline spinner, keep label width stable (see `components.md`). Never let a button look idle while a request is in flight.

## 2. Empty State — never a blank screen

An empty table is an opportunity, not a dead end. Icon + one-line explanation + primary CTA.

```tsx
function EmptyState({ onCreate }: { onCreate: () => void }) {
  return (
    <div className="flex flex-col items-center justify-center rounded-xl border border-dashed bg-card px-6 py-16 text-center">
      <div className="flex size-12 items-center justify-center rounded-full bg-muted">
        <Users className="size-6 text-muted-foreground" />
      </div>
      <h3 className="mt-4 text-sm font-semibold">No patients yet</h3>
      <p className="mt-1 max-w-sm text-sm text-muted-foreground">
        Add your first patient to start tracking visits and reminders.
      </p>
      <Button onClick={onCreate} className="mt-5">
        <Plus className="size-4" /> Add patient
      </Button>
    </div>
  );
}
```

Distinguish **empty (no data ever)** from **no results (filtered)** — the latter offers a "Clear filters" action instead of "Create."

## 3. Error State — visible, recoverable

Never fail silently. For a whole view that failed to load, an inline error card with **Retry**:

```tsx
function ErrorState({ onRetry, message }: { onRetry: () => void; message?: string }) {
  return (
    <div className="flex flex-col items-center justify-center rounded-xl border border-destructive/30 bg-destructive/5 px-6 py-16 text-center">
      <div className="flex size-12 items-center justify-center rounded-full bg-destructive/10">
        <AlertTriangle className="size-6 text-destructive" />
      </div>
      <h3 className="mt-4 text-sm font-semibold">Couldn't load data</h3>
      <p className="mt-1 max-w-sm text-sm text-muted-foreground">
        {message ?? "Something went wrong. Check your connection and try again."}
      </p>
      <Button variant="outline" onClick={onRetry} className="mt-5">
        <RefreshCw className="size-4" /> Retry
      </Button>
    </div>
  );
}
```

- **Field-level errors**: red text under the field + red ring (see form `Field` in `components.md`).
- **Mutation errors**: an error **toast** (below) — the user took an action and deserves immediate feedback.
- Wrap risky subtrees in an **error boundary** so one broken component doesn't blank the whole app.

## 4. Success & Mutation Feedback — toasts (sonner)

Every mutation (create/update/delete) confirms with a toast. Never leave the user guessing whether it worked.

```tsx
// once, in the shell:
import { Toaster } from "sonner";
<Toaster richColors closeButton position="bottom-right" />

// on mutations:
import { toast } from "sonner";
toast.success("Patient saved");
toast.error("Couldn't save patient", { description: err.message });
```

### Optimistic updates (where safe)

Apply the change immediately, roll back on failure — feels instant.

```tsx
async function toggleArchive(patient) {
  const prev = patients;
  setPatients((p) => p.map((x) => x.id === patient.id ? { ...x, archived: !x.archived } : x));
  try {
    await api.update(patient.id, { archived: !patient.archived });
    toast.success(patient.archived ? "Restored" : "Archived");
  } catch (e) {
    setPatients(prev);                       // rollback
    toast.error("Update failed", { description: String(e) });
  }
}
```

## 5. Deletion Confirmation — the most important pattern

**Never use `window.confirm()` or `alert()`.** They're unstyled, jarring, and ignore dark mode. Use a Radix `AlertDialog`.

```tsx
function DeleteDialog({ open, onOpenChange, itemName, onConfirm }: Props) {
  const [pending, setPending] = useState(false);
  async function handle() {
    setPending(true);
    try { await onConfirm(); onOpenChange(false); }
    catch (e) { toast.error("Delete failed", { description: String(e) }); }
    finally { setPending(false); }
  }
  return (
    <AlertDialog open={open} onOpenChange={onOpenChange}>
      <AlertDialogContent>
        <AlertDialogHeader>
          <AlertDialogTitle>Delete {itemName}?</AlertDialogTitle>
          <AlertDialogDescription>
            This permanently removes <span className="font-medium text-foreground">{itemName}</span> and
            all associated records. This action cannot be undone.
          </AlertDialogDescription>
        </AlertDialogHeader>
        <AlertDialogFooter>
          <AlertDialogCancel>Cancel</AlertDialogCancel>
          <AlertDialogAction
            onClick={(e) => { e.preventDefault(); handle(); }}
            disabled={pending}
            className="bg-destructive text-white hover:bg-destructive/90"
          >
            {pending && <Loader2 className="size-4 animate-spin" />}
            Delete
          </AlertDialogAction>
        </AlertDialogFooter>
      </AlertDialogContent>
    </AlertDialog>
  );
}
```

Deletion rules:

- **Title states the consequence** ("Delete invoice #1042?"), not a generic "Are you sure?".
- **Body names the specific item** and says whether it's reversible.
- **Destructive button is red**; Cancel is the default/safe choice and gets autofocus.
- **Disable + spinner** while the delete is in flight.

### Stronger guards for dangerous deletes

For irreversible or bulk deletes, **require typing the name** to confirm:

```tsx
const [confirmText, setConfirmText] = useState("");
// …
<Input value={confirmText} onChange={(e) => setConfirmText(e.target.value)}
       placeholder={`Type "${itemName}" to confirm`} />
<AlertDialogAction disabled={confirmText !== itemName || pending} … >Delete</AlertDialogAction>
```

### Prefer soft delete + Undo (best UX where the domain allows)

Instead of a hard delete + confirmation friction, archive the row and offer **Undo** in a toast. Safer and smoother — the user isn't punished for misclicks, and you avoid the modal entirely for reversible actions.

```tsx
function archivePatient(patient) {
  const prev = patients;
  setPatients((p) => p.filter((x) => x.id !== patient.id));   // optimistic remove
  const t = setTimeout(() => api.archive(patient.id), 5000);  // commit after grace period
  toast("Patient archived", {
    action: {
      label: "Undo",
      onClick: () => { clearTimeout(t); setPatients(prev); },
    },
  });
}
```

(The Shine project already does soft delete with archive/restore — this pattern fits it directly.)

## State Coverage Checklist (per async view)

- [ ] Loading → skeleton matching final layout
- [ ] Empty (no data) → icon + message + create CTA
- [ ] No results (filtered) → message + clear-filters CTA
- [ ] Error → inline card with Retry
- [ ] Mutation success → toast
- [ ] Mutation error → toast with cause, state rolled back
- [ ] Delete → AlertDialog (or soft-delete + Undo), red action, pending spinner
- [ ] Button-in-flight → disabled + spinner, stable width
