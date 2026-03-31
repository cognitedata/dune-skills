---
name: performance
description: Optimize a Dune app for speed, rendering efficiency, CDF query performance, and bundle size. Use this skill whenever the user mentions slow load times, lag, excessive re-renders, large bundles, slow CDF queries, long lists, virtualization, code splitting, or React Profiler results — even if they just say "it feels slow" without naming a specific cause.
---

# Performance Optimization

Systematically identify and fix performance issues in **$ARGUMENTS** (or the whole
app if no argument is given).

## Why This Matters

Performance issues compound — a 200ms CDF query plus an unnecessary re-render plus
an unvirtualized list adds up to a UI that feels broken. Users don't diagnose which
layer is slow; they just leave. Dune apps are especially susceptible because CDF
responses can be large, deeply nested, and slow on first load. A methodical approach
(measure → identify → fix → re-measure) prevents wasted effort on optimizations
that don't move the needle.

---

## Step 1 — Establish a Baseline

Before changing any code, capture current performance numbers so every optimization
can be measured against them.

**What the agent can do:**

```bash
pnpm run build 2>&1
```

Note the build time and output chunk sizes from the build log. If there's a
`rollup-plugin-visualizer` already configured, inspect the generated treemap.

**Ask the user to do:**

Tell the user: "Before I start making changes, it helps to have baseline numbers.
Can you open the app in Chrome and capture a Lighthouse Performance score and a
React Profiler recording of the slow interaction? I'll use those to prioritize
what to fix first."

If the user provides profiler data or specific complaints (e.g., "the asset table
takes 3 seconds to filter"), use that to focus the review. If they don't have
profiler data, proceed with static analysis — the code patterns below reliably
correlate with performance problems.

---

## Step 2 — Identify Unnecessary Re-renders

Re-renders are the most common performance issue in React apps. A component that
re-renders when its output hasn't changed wastes CPU and can cascade to its entire
subtree.

Read the component tree starting from `src/App.tsx` and search for these patterns:

**Inline object/array/function creation in JSX** — creates a new reference on every
render, which breaks shallow equality checks and forces child components to
re-render even when the data hasn't changed:

```tsx
// Problem: new object every render → Chart re-renders every time
<Chart options={{ color: "red" }} />

// Fix: stable reference
const chartOptions = useMemo(() => ({ color: "red" }), []);
<Chart options={chartOptions} />
```

**Event handlers recreated on every render:**

```tsx
// Problem: new function reference every render
<Button onClick={() => doSomething(id)} />

// Fix: stable callback
const handleClick = useCallback(() => doSomething(id), [id]);
<Button onClick={handleClick} />
```

**Context providers with unstable values** — every consumer re-renders when the
context value's reference changes, even if the actual data is the same:

```tsx
// Problem: new object reference every render → all consumers re-render
<MyContext.Provider value={{ user, sdk }}>

// Fix: memoize the value
const ctxValue = useMemo(() => ({ user, sdk }), [user, sdk]);
<MyContext.Provider value={ctxValue}>
```

Search for these patterns:

```bash
rg -n "value=\{\{|onClick=\{\(\)" --type ts src/
```

Apply `React.memo` only to components confirmed to re-render unnecessarily — wrapping
everything in `memo` adds overhead from the shallow comparison and can mask real bugs
where props *should* trigger a re-render.

---

## Step 3 — Optimize CDF Data Fetching

CDF queries are often the single largest contributor to perceived slowness. A query
that fetches too much data or runs too often dominates the user experience far more
than any rendering optimization.

Search for CDF SDK calls:

```bash
rg -n "sdk\.|client\.|useQuery|useCogniteClient" --type ts src/
```

For each call, check:

| Issue | Why it matters | Fix |
|-------|---------------|-----|
| No `limit` set | CDF returns up to 1000 items by default — far more than most UIs need | Add `limit: 100` or the actual page size |
| Fetching all properties | CDF instances can have dozens of properties per item | Add a `sources`/`properties` filter for only the fields you display |
| Fetching on every render | Missing dependency array or unstable deps re-trigger the query constantly | Move into `useQuery` with a stable `queryKey` |
| Sequential requests that could be parallel | Two independent fetches that wait for each other double the wall-clock time | Use `Promise.all` or `useQueries` |
| Search input without debounce | Every keystroke fires a CDF query, most of which are discarded | Add 300–500ms debounce before firing the query |

**Example** — scoped instance query:

```ts
// Problem: fetches everything
const result = await client.instances.list({ instanceType: "node" });

// Fix: scoped, limited, and filtered
const result = await client.instances.list({
  instanceType: "node",
  sources: [{
    source: { type: "view", space: "my-space", externalId: "Asset", version: "v1" },
  }],
  filter: { equals: { property: ["node", "space"], value: "my-space" } },
  limit: 100,
});
```

---

## Step 4 — Virtualize Large Lists

Rendering hundreds of DOM nodes for a long list forces the browser to layout and
paint elements the user can't see. This causes visible jank on scroll and slow
initial renders.

```bash
rg -n "\.(map|forEach)\(" --type ts src/
```

For any list where the data source could exceed ~50 items, replace the plain
`.map()` render with a virtualized list. Use `@tanstack/react-virtual`:

```tsx
import { useVirtualizer } from "@tanstack/react-virtual";

const rowVirtualizer = useVirtualizer({
  count: items.length,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 48,
});

return (
  <div ref={parentRef} style={{ height: "600px", overflow: "auto" }}>
    <div style={{ height: rowVirtualizer.getTotalSize() }}>
      {rowVirtualizer.getVirtualItems().map((virtualRow) => (
        <div
          key={virtualRow.key}
          style={{ transform: `translateY(${virtualRow.start}px)` }}
        >
          {items[virtualRow.index].name}
        </div>
      ))}
    </div>
  </div>
);
```

---

## Step 5 — Code-Split Heavy Pages and Components

Any page or modal not needed on initial load should be lazy-loaded. This directly
reduces the JavaScript the browser must parse before the app becomes interactive.

Read the router setup and identify routes that are statically imported but not
shown on the landing page:

```tsx
// Before: imported at module level, included in initial bundle
import { ReportPage } from "./pages/ReportPage";

// After: loaded on demand when the user navigates to this route
import { lazy, Suspense } from "react";
const ReportPage = lazy(() => import("./pages/ReportPage"));

// In the route:
<Suspense fallback={<PageSkeleton />}>
  <ReportPage />
</Suspense>
```

Also check for large third-party imports (chart libraries, PDF viewers, map
renderers) that are imported at the module level but only used inside a specific
component. Move these to dynamic imports inside the component that needs them.

---

## Step 6 — Analyse and Reduce Bundle Size

Run a production build and inspect what's in the bundle:

```bash
pnpm run build 2>&1
```

If the app uses Vite, temporarily add the visualizer to see a treemap:

```bash
pnpm add -D rollup-plugin-visualizer
```

```ts
// vite.config.ts — add temporarily, revert after analysis
import { visualizer } from "rollup-plugin-visualizer";

export default defineConfig({
  plugins: [
    react(),
    visualizer({ open: true, gzipSize: true, brotliSize: true }),
  ],
});
```

Run `pnpm run build` and inspect the treemap. Flag any chunk over 100 KB gzipped
that isn't a core dependency.

Common wins:

| Bloated dependency | Replacement |
|-------------------|-------------|
| `lodash` (full import) | Individual `lodash-es/` imports or native equivalents |
| `moment` | `date-fns` or native `Intl.DateTimeFormat` |
| Full chart library imported | Tree-shaken named imports (e.g., ECharts `use([...])`) |

Revert the visualizer plugin after analysis.

---

## Step 7 — Check for Memory Leaks

Memory leaks cause the app to slow down over time as the user navigates between
views. The most common source is `useEffect` hooks that set up subscriptions or
async work without returning a cleanup function.

```bash
rg -n "useEffect" --type ts -A 10 src/
```

For each `useEffect`, verify:

- `addEventListener` has a matching `removeEventListener`
- `setInterval` / `setTimeout` has a matching `clearInterval` / `clearTimeout`
- Async fetches use an `AbortController` that is aborted in the cleanup
- Store subscriptions call the returned unsubscribe function

Also search for event listeners attached outside of React:

```bash
rg -n "addEventListener" --type ts src/
```

If any are attached at module level or in a non-React utility, verify they're
removed when no longer needed.

---

## Step 8 — Measure After and Report

Use this exact template for the report:

### Performance Review — [App/Component Name]

**Summary**: [1-2 sentence overview of what was found and the highest-impact fix]

**Baseline** (captured before changes):

| Metric | Value |
|--------|-------|
| Build time | |
| Bundle size (gzipped) | |
| Lighthouse Performance (if provided) | |
| Specific metric from user complaint | |

**Changes made:**

| # | File | Change | Expected Impact |
|---|------|--------|-----------------|
| 1 | `path/to/file` | What was changed | Why it helps |
| 2 | `path/to/file` | What was changed | Why it helps |

**After** (measured post-changes):

| Metric | Before | After | Delta |
|--------|--------|-------|-------|
| Build time | | | |
| Bundle size (gzipped) | | | |

### Steps with no issues found
- [List any steps where no issues were found]

If a metric couldn't be measured (e.g., the user didn't provide Lighthouse data),
state that explicitly rather than guessing.

After presenting the report, ask the user to re-run Lighthouse and the React
Profiler to confirm the improvements.

---

## Gotchas

1. **Don't optimize without measuring.** A `useMemo` on a cheap computation adds
   overhead from the dependency comparison. The profiler tells you where time is
   actually spent — optimize those spots, not everything that looks suboptimal.

2. **`React.memo` is not free.** It adds a shallow comparison on every render. For
   components that always receive new props (e.g., children from a parent that
   re-renders with new data), `memo` makes things slower, not faster.

3. **Virtualization changes accessibility.** Screen readers may not be able to
   navigate all items in a virtualized list. If the list needs to be accessible,
   add `aria-rowcount` and `aria-rowindex` attributes.

4. **Code splitting adds loading states.** Every `lazy()` boundary needs a
   `Suspense` fallback. If the fallback is jarring (e.g., a spinner replacing
   a fully-rendered page), the perceived performance can feel worse even though
   the initial load is faster.

5. **CDF query limits interact with pagination.** Setting `limit: 100` doesn't
   help if the component then makes 10 sequential cursor-based requests to fetch
   all 1000 items. If you need all the data, paginate in the UI instead.

6. **Tree-shaking requires ESM imports.** `import _ from "lodash"` pulls in the
   entire library. `import groupBy from "lodash-es/groupBy"` allows the bundler
   to tree-shake the rest. Check that imports use the `-es` variant or named
   imports from an ESM-compatible package.
