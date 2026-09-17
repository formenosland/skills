# Frontend

State lives in more places than people track: component state, context, store, URL, cache, localStorage, form state, server state. Drift between any two is a bug.

## Query cache keys

If the codebase uses TanStack Query / SWR / Apollo / RTK Query:

- **Centralize query keys in a single registry per resource.** Two components, two keys for the same resource → guaranteed stale cache after a mutation.
  ```ts
  export const userKeys = {
    all: ['users'] as const,
    lists: () => [...userKeys.all, 'list'] as const,
    list: (f: Filters) => [...userKeys.lists(), f] as const,
    details: () => [...userKeys.all, 'detail'] as const,
    detail: (id: string) => [...userKeys.details(), id] as const,
  };
  ```
- **Invalidation must match the registry.** `invalidateQueries({ queryKey: userKeys.detail(id) })`, not `['user', id]`. Mismatch → invalidation silently does nothing.
- **Optimistic updates need a rollback path.** `onError` restores the previous snapshot, or the UI lies persistently.
- **One fetch per resource per page**, not one per component. Three components wanting the same user → one hook fetches, others read from cache.

## Notifications (toasts)

Toasts / snackbars / ephemeral messages belong at the **app shell**, one provider at the root. Feature-local `MessageService` / `ToastProvider` providers:

- lose toasts on navigation (provider unmounts),
- split styling,
- complicate tests.

A diff that adds a local toast provider in a feature is a blocker — route to the root host.

## Lifecycle paradigms don't mix

Pick one reactive model per component:

- **React** — hooks, not class lifecycle. Don't mix.
- **Angular** — with signal inputs (`input()`), react via `effect()` / `computed()`. Not `ngOnChanges`. Mixing produces inconsistent timing: some values update synchronously, others on the next change-detection pass.
- **Vue** — `watch` / `watchEffect` / `computed` / lifecycle hooks each have a place, but deep interdependencies across them cascade unpredictably.

## Component size & responsibility

A component hitting:

- 500+ lines of logic/template,
- multiple concerns (fetch + mutate + render + upload + dialogs + forms),
- many loosely-related `useState` calls,

is a refactor waiting. Split: container (owns fetch/mutations), presentational children, separate dialogs, custom hooks for complex state transitions. Flag when the diff adds further to an already-bloated component.

## State location

- **Server state** (API data) → query cache. Never duplicated into `useState`.
- **URL state** (filters, search, tab, page) → the URL. Bookmarkable, refresh-safe.
- **Form state** → form library hook (`react-hook-form`, `formik`, `@tanstack/react-form`). Not `useState` per field.
- **Ephemeral UI state** (open/closed, hover) → component-local `useState`.
- **Cross-component UI state** → the store (Context, Zustand, Redux — whatever the codebase uses).

Server data in `useState` is a red flag: refetch, invalidation, cache-sharing, refresh semantics are now all hand-rolled and usually wrong.

## Derived state

If a value is a function of other state, **compute it** — don't store it. In React specifically: `useEffect` to sync one state to another based on a prop change is almost always a bug. Compute inline or `useMemo`.

```ts
// ❌ two sources of truth, drift inevitable
const [users, setUsers] = useState(...);
const [adminCount, setAdminCount] = useState(0);

// ✅ one source, derived
const [users, setUsers] = useState(...);
const adminCount = users.filter(u => u.role === 'admin').length;
```

## UI options from schema

Enums, statuses, role names, type literals: import from the shared schema (Zod / OpenAPI-derived types / protobuf). A client-side re-declaration of server enums will drift the first time anyone adds a value.

Drift symptom: backend adds `banned`, frontend dropdown doesn't know, filter lists in five places each miss different subsets of statuses. Fix: one source, referenced everywhere.

## Forms

- **One validation source** — shared schema (Zod works both sides; OpenAPI → client validators). Duplicated rules drift.
- **Controlled vs uncontrolled** — choose per form, don't mix within one form.
- **Disable submit while in flight** _and_ guard the mutation itself (in-flight flag or idempotency key) — disabled-button-only protection fails on double-click.
- **Per-field error state**, not one generic banner.
- **Preserve field state on failure.** Don't clear 20 fields because the server said no.

## Accessibility

- Semantic HTML: `<button>`, `<a>`, `<label>`, `<input type="checkbox">`. Not div-with-role.
- Keyboard nav on everything interactive; Enter/Space activates; sensible tab order.
- Focus management: dialog opens → focus moves in; closes → focus returns to trigger.
- ARIA only when semantic HTML doesn't suffice. Don't `aria-label` a `<button>` with its own text.
- Color contrast WCAG AA; never color-alone to convey meaning (pair with icon/label).
- Respect `prefers-reduced-motion`.

## Performance

- **Measure before optimizing.** Don't `useMemo` / `useCallback` reflexively.
- Virtualize lists over ~500 rows (`react-window`, `@tanstack/virtual`).
- Images: explicit dimensions + `loading="lazy"` unless above the fold.
- Route-level code-splitting as a baseline.
- Bundle size is a review concern — a 200KB dependency needs justification.

## SSR / hydration

- Server and client must agree on first render. Reading `window` / `localStorage` / `Date.now()` / random during render → hydration mismatches (flicker, lost event handlers).
- Gate time-dependent or random rendering on "have we hydrated yet?" (`useEffect` + flag).

## Error boundaries

- At least one per route. Otherwise a leaf error takes down the app.
- **Don't catch async errors / promise rejections / event-handler errors** — boundaries don't see those. Handle at the call site.
- Boundary fallback also logs to observability. A silent "Something went wrong" is a bug blackhole.

## Client-side observability

- Error monitoring (Sentry / Datadog RUM / equivalent) for uncaught exceptions and promise rejections.
- No PII in client logs (same as server).
- Source maps uploaded to the monitoring tool, not publicly shipped.
