---
name: performance
description: "MUST be used whenever optimizing a Dune app for speed, reducing render counts, improving CDF query efficiency, or reducing bundle size. Do NOT skip measurement steps — always profile before changing code. Triggers: performance, slow, laggy, optimize, optimization, re-render, bundle size, load time, Lighthouse, profiler, virtualization, lazy load, code split, CDF query, large list, memory leak."
allowed-tools: Read, Glob, Grep, Shell, Write
metadata:
  argument-hint: "[file, component, or area to optimize — e.g. 'src/components/AssetTable.tsx']"
---

# Performance Optimization

Systematically identify and fix performance issues in **$ARGUMENTS** (or the whole app if no argument is given). Always measure first — never optimize blindly.

---

## Step 1 — Measure baseline before touching anything

Run the production build and capture metrics before making any changes:

```bash
pnpm run build
pnpm run preview
```

Open the app in Chrome and capture:
- **Lighthouse score** (Performance tab → Run audit)
- **React Profiler** (React DevTools → Profiler → Record an interaction)
  - Note the components with the longest render times and highest render counts

Record baseline numbers. Every optimization must be measured against these.

---

## Step 2 — Identify unnecessary re-renders

Read the component tree (start from `src/App.tsx`) and look for these patterns:

**Inline object/array/function creation in JSX:**
```tsx
// BAD — new object on every render causes children to re-render
<Chart options={{ color: "red" }} />

// GOOD
const chartOptions = useMemo(() => ({ color: "red" }), []);
<Chart options={chartOptions} />
```

**Event handlers recreated on every render:**
```tsx
// BAD
<Button onClick={() => doSomething(id)} />

// GOOD
const handleClick = useCallback(() => doSomething(id), [id]);
<Button onClick={handleClick} />
```

**Context that changes on every render:**
```tsx
// BAD — new object reference every render
<MyContext.Provider value={{ user, sdk }}>

// GOOD — memoize the context value
const ctxValue = useMemo(() => ({ user, sdk }), [user, sdk]);
<MyContext.Provider value={ctxValue}>
```

Search for common culprits:
```bash
grep -rn --include="*.tsx" \
  -E "value=\{\{|onClick=\{\(\)" src/
```

Apply `React.memo` to pure presentational components that receive stable props. Do NOT wrap every component — only those confirmed to re-render unnecessarily via the Profiler.

---

## Step 3 — Optimize CDF data fetching

Read all CDF SDK calls (search for `sdk.`, `client.`, `useQuery`, `useCogniteClient`).

For each call, check:

| Issue | Fix |
|-------|-----|
| No `limit` set | Add `limit: 100` (or the actual page size needed) |
| Fetching all properties | Add a `properties` filter to select only required fields |
| Fetching on every render | Move inside `useQuery`/`useMemo` with a stable dependency array |
| Sequential requests that could be parallel | Use `Promise.all` or batched SDK methods |
| Polling without debounce | Add debounce (300–500 ms) for search inputs before firing CDF queries |

Example — scoped instance query:
```ts
// BAD — fetches everything
const result = await client.instances.list({ instanceType: "node" });

// GOOD — scoped and limited
const result = await client.instances.list({
  instanceType: "node",
  sources: [{ source: { type: "view", space: "my-space", externalId: "Asset", version: "v1" } }],
  filter: { equals: { property: ["node", "space"], value: "my-space" } },
  limit: 100,
});
```

---

## Step 4 — Virtualize large lists

Search for lists that render more than ~50 items:
```bash
grep -rn --include="*.tsx" -E "\.(map|forEach)\(" src/
```

For any list where the data source could exceed 50 items, replace the plain `.map()` render with a virtualized list. Use `@tanstack/react-virtual` (already a Dune ecosystem dependency):

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

## Step 5 — Code-split heavy pages and components

Any page or modal that is not needed on the initial load should be lazy-loaded. Read the router setup and identify routes that are imported statically but not shown on the landing page.

```tsx
// BEFORE
import { ReportPage } from "./pages/ReportPage";

// AFTER
import { lazy, Suspense } from "react";
const ReportPage = lazy(() => import("./pages/ReportPage"));

// In the route:
<Suspense fallback={<PageSkeleton />}>
  <ReportPage />
</Suspense>
```

Similarly, large third-party components (chart libraries, PDF viewers, map renderers) should be dynamically imported inside the component that needs them, not at the module level.

---

## Step 6 — Analyse and reduce bundle size

```bash
# Install if not already present, then run
pnpm add -D rollup-plugin-visualizer
```

Add to `vite.config.ts` temporarily:
```ts
import { visualizer } from "rollup-plugin-visualizer";

export default defineConfig({
  plugins: [
    react(),
    visualizer({ open: true, gzipSize: true, brotliSize: true }),
  ],
});
```

Run `pnpm run build` and inspect the treemap. Flag any chunk > 100 KB (gzipped) that is not a necessary initial dependency.

Common wins:
- Replace `lodash` with individual `lodash-es` imports or native equivalents
- Replace `moment` with `date-fns` or native `Intl`
- Ensure chart libraries (e.g., ECharts, Recharts) are tree-shaken correctly

Revert the visualizer plugin after analysis.

---

## Step 7 — Fix memory leaks

Search for `useEffect` hooks that set up subscriptions, timers, or event listeners without cleanup:

```bash
grep -rn --include="*.tsx" --include="*.ts" -A 10 "useEffect" src/
```

Every `useEffect` that calls `addEventListener`, `setInterval`, `setTimeout`, `subscribe`, or sets up a CDF streaming connection must return a cleanup function:

```ts
useEffect(() => {
  const controller = new AbortController();
  fetchData(controller.signal);
  return () => controller.abort();
}, [id]);
```

---

## Step 8 — Measure after and report

Re-run the same Lighthouse audit and React Profiler session from Step 1. Report the delta:

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Lighthouse Performance | 72 | 91 | +19 |
| Largest Contentful Paint | 3.2 s | 1.8 s | −1.4 s |
| Total Blocking Time | 420 ms | 80 ms | −340 ms |
| Bundle size (gzipped) | 410 KB | 290 KB | −120 KB |
| `AssetTable` render count (on filter change) | 8 | 2 | −6 |

If a step produced no improvement, state that explicitly. Do not fabricate numbers.

---

## Done

List every change made with the file path and a one-line explanation. If further gains require server-side or infrastructure changes (e.g., CDF response caching, CDN configuration), note them separately as out-of-scope recommendations.
