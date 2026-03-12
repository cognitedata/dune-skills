---
name: correctness-and-error-handling
description: Review a Dune app for correctness issues, missing error handling, and unhandled edge cases. Use this skill whenever the user asks for a bug check, correctness review, error handling audit, or robustness review. Also use it when they mention crashes, null/undefined errors, missing loading states, empty states, unhandled promises, useEffect cleanup, runtime type safety, or async errors — even if they don't frame it as a "review." If the user reports a crash or blank screen, this skill likely covers the root cause.
---

# Correctness & Error Handling Review

Review **$ARGUMENTS** (or the whole app if no argument is given) for correctness
issues and missing error handling. Work through every section below and report
findings with file paths and line numbers.

## Why This Matters

A single unhandled null dereference or missing error state can crash the entire
app in production. CDF responses are external data — they can be empty, malformed,
or fail entirely. Users on flaky networks will hit every unhappy path. This review
catches the issues that only surface when real-world conditions diverge from the
happy path.

---

## Step 1 — Map Data Flows

Read these files first to build a mental model of how data moves through the app:

- `src/main.tsx` / `src/App.tsx` — top-level error boundaries and auth flow
- All files matching `**/hooks/*.ts`, `**/contexts/*.tsx` — shared async state
- All files matching `**/api/*.ts`, `**/services/*.ts` — CDF SDK call sites

For each async data source, note:

| Question | Why it matters |
|----------|---------------|
| What happens when the request fails? | Network errors, CDF 403, timeouts are common in production |
| What does the UI show while loading? | Users staring at a blank screen assume the app is broken |
| What does the UI show if the result is empty? | An empty `<div>` is indistinguishable from a bug |

---

## Step 2 — Verify Error Boundaries

An unhandled render-time exception without an Error Boundary produces a blank white
screen with no recovery path — the user has to refresh the page and loses any
in-progress work.

Search for existing error boundaries:

```bash
rg -n "ErrorBoundary|componentDidCatch|getDerivedStateFromError" --type ts src/
```

If none exist, add one around the main app content:

```tsx
import { ErrorBoundary } from "react-error-boundary";

function ErrorFallback({ error }: { error: Error }) {
  return (
    <div role="alert" className="p-8 text-center">
      <p className="text-lg font-semibold">Something went wrong</p>
      <pre className="mt-2 text-sm text-muted-foreground">{error.message}</pre>
    </div>
  );
}

// In App.tsx:
<ErrorBoundary FallbackComponent={ErrorFallback}>
  <MainContent />
</ErrorBoundary>
```

---

## Step 3 — Audit Async Error Handling

Unhandled promise rejections silently swallow errors. The user sees no feedback —
the UI just stops responding or shows stale data. This is the most common class of
bug in Dune apps because CDF calls are async and fail more often than developers
expect (auth expiry, rate limits, network drops).

```bash
rg -n "async\s+function|async\s+\(" --type ts src/
rg -n "\.then\(" --type ts src/
```

For each hit, verify errors are caught and surfaced. The correct pattern depends
on the data-fetching approach:

**Direct async functions** — wrap in try/catch, re-throw so callers can handle:

```ts
async function fetchAssets(sdk: CogniteClient) {
  try {
    const result = await sdk.assets.list({ limit: 100 });
    return result.items;
  } catch (error) {
    console.error("Failed to fetch assets:", error);
    throw error;
  }
}
```

**TanStack Query** — the library catches errors, but the component must consume
`isError` and `error`. Without this, a failed request shows stale data or nothing:

```tsx
const { data, isLoading, isError, error } = useQuery({
  queryKey: ["assets"],
  queryFn: () => fetchAssets(sdk),
});

if (isError) return <ErrorMessage error={error} />;
```

**`.then()` chains** — flag any `.then()` without a `.catch()`:

```ts
// Problem: silent failure
sdk.assets.list().then((res) => setAssets(res.items));

// Fix
sdk.assets.list()
  .then((res) => setAssets(res.items))
  .catch((err) => setError(err));
```

---

## Step 4 — Loading, Error, and Empty States

Every component that fetches data needs three distinct UI paths. Without them, the
user can't tell whether data is loading, failed, or simply empty — all three look
the same (a blank area).

| State | What to render |
|-------|---------------|
| Loading | Spinner, skeleton, or progress indicator |
| Error | Human-readable message (not a raw error object or blank space) |
| Empty | "No results" / "Nothing here yet" (not an empty container) |

Search for components that render data without guarding:

```bash
rg -n "\.(map|filter|find)\(" --type ts src/
```

Read each component and verify loading/error/empty states are handled before the
`.map()` call. A common miss: `data?.map(...)` handles `undefined` but still
renders nothing for `[]`.

---

## Step 5 — Validate External Data at Runtime

TypeScript types are compile-time only — they evaporate at runtime. Data from CDF
responses, URL params, `localStorage`, and `JSON.parse` can be anything regardless
of type annotations. Casting with `as MyType` is especially risky because it
silences the compiler while providing zero runtime protection — the app compiles
cleanly but crashes in production when the data doesn't match.

```bash
rg -n "JSON\.parse\(" --type ts src/
rg -n "localStorage\.(get|set)Item" --type ts src/
rg -n "useSearchParams|searchParams\.get" --type ts src/
```

For each hit, verify the value is validated before use:

| Approach | Example |
|----------|---------|
| **Zod schema** | `const parsed = MySchema.safeParse(raw);` |
| **Type guard** | `if (typeof value !== "string") return;` |
| **Nullish fallback** | `const id = params.get("id") ?? defaultId;` |

Flag any `as SomeType` cast on external data and recommend replacing it with
runtime validation.

---

## Step 6 — Null, Undefined, and Empty Arrays

CDF responses frequently contain optional fields, deeply nested properties, and
arrays that can be empty or missing entirely. Direct property access without
guarding is the single most common source of runtime crashes in Dune apps.

```bash
rg -n "\w+\[0\]\." --type ts src/
```

Patterns to flag and fix:

```tsx
// Crashes if data is undefined
const name = asset.properties.space.Asset.name;
// Fix: optional chaining with fallback
const name = asset.properties?.["my-space"]?.["Asset"]?.name ?? "Unknown";

// Crashes if items is undefined
items.map(renderItem);
// Fix: default to empty array
(items ?? []).map(renderItem);

// Crashes if the array is empty
const first = items[0].name;
// Fix: safe access
const first = items.at(0)?.name ?? "—";
```

---

## Step 7 — useEffect Cleanup

A `useEffect` that sets up a subscription, timer, or async operation without
returning a cleanup function causes memory leaks and "setState on unmounted
component" bugs. These rarely appear in development — they surface in production
when users navigate between views quickly, causing the previous view's async work
to complete after the component is gone.

```bash
rg -n "useEffect" --type ts -A 15 src/
```

For each `useEffect`, check for the matching cleanup:

| Setup pattern | Required cleanup |
|---------------|-----------------|
| `addEventListener` | `removeEventListener` |
| `setInterval` / `setTimeout` | `clearInterval` / `clearTimeout` |
| CDF streaming / SSE | Close the stream |
| `fetch` / CDF SDK call | `AbortController.abort()` |
| Event emitter / store subscription | Call the returned unsubscribe function |

**Example** — async fetch with abort handling:

```ts
useEffect(() => {
  const controller = new AbortController();

  async function load() {
    try {
      const data = await fetchWithSignal(controller.signal);
      if (!controller.signal.aborted) setState(data);
    } catch (err) {
      if (err instanceof Error && err.name !== "AbortError") {
        setError(err);
      }
    }
  }

  load();
  return () => controller.abort();
}, [id]);
```

---

## Step 8 — Race Conditions and Stale State

Async operations that update state can race with each other, producing stale or
out-of-order results. This is especially common in search-as-you-type flows and
navigation-triggered fetches where the user acts faster than the network responds.

**Stale closures in useState** — an async callback captures state from render N
but completes during render N+1, overwriting newer state:

```tsx
// Problem: response from old query overwrites newer result
const [items, setItems] = useState([]);
useEffect(() => {
  fetchItems(query).then((result) => setItems(result));
}, [query]);

// Fix: abort the previous request so only the latest writes
useEffect(() => {
  const controller = new AbortController();
  fetchItems(query, controller.signal).then((result) => {
    if (!controller.signal.aborted) setItems(result);
  });
  return () => controller.abort();
}, [query]);
```

Also check for concurrent mutations — if the user can trigger a new action before
the previous completes (e.g. clicking "Save" twice), verify the app either
debounces/disables the action or handles the overlap gracefully.

---

## Step 9 — React Key Props

Missing or unstable `key` props on list items cause subtle rendering bugs — stale
component state, broken animations, and input fields that lose their value when the
list reorders. These bugs are hard to spot because the initial render looks correct.

```bash
rg -n "\.map\(" --type ts src/
```

For each `.map()` that returns JSX, verify:

- A `key` prop is present on the outermost returned element
- The key is **stable and unique** — use `item.id` or another persistent identifier
- Array index (`i`) is only acceptable for static lists that never reorder, filter, or grow

```tsx
// Bad: index key on a dynamic list
{items.map((item, i) => <Card key={i} item={item} />)}

// Good: stable ID
{items.map((item) => <Card key={item.id} item={item} />)}
```

---

## Step 10 — Edge Cases

For each feature, think through boundary conditions that are easy to miss during
development but common in production:

| Case | What to verify |
|------|----------------|
| **Empty data** | Zero-item lists, zero-result CDF queries |
| **Single item** | List with exactly one entry (common off-by-one source) |
| **Maximum data** | CDF returns full `limit` — is pagination or "showing N of M" communicated? |
| **Network offline** | Meaningful message shown, not silent failure |
| **Auth expiry** | CDF returns 401/403 mid-session — does the app redirect to login or show an error? |

For Atlas tool `execute` functions, validate every `args` field before calling CDF:

```ts
execute: async (args) => {
  if (!args.assetId || typeof args.assetId !== "string") {
    return { output: "Missing or invalid assetId", details: null };
  }
  // safe to proceed
}
```

---

## Report Findings

Use this exact template for the report:

### Correctness Review — [App/Component Name]

**Summary**: [1-2 sentence overview of overall health and most critical issues]

| Severity | File | Line | Issue | Recommended Fix |
|----------|------|------|-------|-----------------|
| HIGH | `path/to/file` | N | Description | How to fix |
| MEDIUM | `path/to/file` | N | Description | How to fix |
| LOW | `path/to/file` | N | Description | How to fix |

### Steps with no issues found
- [List any steps where no issues were found]

HIGH = crashes, data loss, or misleading UI in production.
MEDIUM = degraded UX (missing loading/empty states, poor error messages).
LOW = code quality or minor robustness improvements.

If no issues are found in a step, say so explicitly — skipping steps silently
makes it unclear whether the step was checked.

After presenting the report, offer to fix the HIGH and MEDIUM issues directly.

---

## Gotchas

1. **Don't trust `.length` checks alone.** `if (data.length)` crashes if `data`
   is `undefined`. Guard with `data?.length` or `Array.isArray(data) && data.length`.

2. **Mixing `?.` and `.` in a chain.** `obj?.foo.bar` is not fully safe — if `foo`
   is `undefined`, the `.bar` access still crashes. Use `obj?.foo?.bar` to guard
   every level. This is easy to miss when adding optional chaining to existing code.

3. **`catch` at the wrong level.** A try/catch inside a utility function catches
   the error but may not surface it to the UI. Verify the catch either re-throws
   or sets error state that a component reads.

4. **Error boundaries don't catch async errors.** React Error Boundaries only
   catch errors during rendering, lifecycle methods, and constructors. They don't
   catch errors in event handlers, `setTimeout`, or async functions — those need
   their own try/catch.

5. **TanStack Query retries mask errors.** By default, `useQuery` retries failed
   requests 3 times before surfacing `isError`. During retries, the user sees
   loading state for much longer than expected. Consider setting `retry: false` or
   `retry: 1` for CDF calls where retrying won't help (auth errors, 404s).

6. **Empty arrays are truthy.** `if (data)` passes for `[]`, so a component that
   checks `if (data)` and then renders `data.map(...)` will render nothing instead
   of an empty state. Check `data.length === 0` explicitly.
