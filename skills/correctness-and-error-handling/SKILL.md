---
name: correctness-and-error-handling
description: "MUST be used whenever reviewing a Dune app for bugs, missing error states, unhandled promise rejections, or incorrect edge-case behaviour. Do NOT skip — run every step when the user asks for a correctness review, bug check, error handling audit, or robustness review. Triggers: correctness, error handling, bug, edge case, crash, unhandled, null, undefined, empty state, loading state, error boundary, try catch, async error, useEffect cleanup, type guard, runtime error, robustness."
allowed-tools: Read, Glob, Grep, Shell, Write
metadata:
  argument-hint: "[file or directory to review, or leave blank to review the whole app]"
---

# Correctness & Error Handling Review

Review **$ARGUMENTS** (or the whole app if no argument is given) for correctness issues and missing error handling. Work through every step below and report all findings with file paths and line numbers.

---

## Step 1 — Map data flows

Read these files before checking anything:

- `src/main.tsx` / `src/App.tsx` — top-level error boundaries and auth flow
- All files matching `**/hooks/*.ts`, `**/contexts/*.tsx` — shared async state
- All files matching `**/api/*.ts`, `**/services/*.ts` — CDF SDK call sites

For each async data source, note:
- What happens when the request fails (network error, CDF 403, timeout)?
- What does the UI show while loading?
- What does the UI show if the result is empty?

### Check for known defects in critical paths

Before proceeding to the detailed checks below, scan for markers of unresolved issues in hooks, services, and API call sites:

```bash
# Find TODO/FIXME/HACK in critical code paths (not test files)
grep -rn --include="*.ts" --include="*.tsx" -E "(TODO|FIXME|HACK|XXX):" src/ | grep -v ".test." | grep -v ".spec."

# Find "fix" or "broken" or "workaround" markers
grep -rn --include="*.ts" --include="*.tsx" -i -E "(TODO.*fix|workaround|broken|known.?bug|temporary.?hack)" src/
```

For each match:
- Is it in a critical path (data fetching, rendering, auth, navigation)?
- Does it indicate a silent failure, data corruption risk, or incorrect behavior?
- Is there an associated test that verifies the workaround or guards against the known issue?

Flag any `TODO: fix` or `FIXME` in critical paths as a **known defect** that must be resolved or explicitly scoped out with a safe failure mode.

---

## Step 2 — Verify top-level error boundaries

Every Dune app must have at least one React Error Boundary wrapping the main content so that an unexpected render-time exception shows a user-friendly message instead of a blank screen.

```bash
grep -rn --include="*.tsx" --include="*.ts" -E "ErrorBoundary|componentDidCatch|getDerivedStateFromError" src/
```

If no error boundary exists, add one around the main app content:

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

## Step 3 — Audit async functions for missing try/catch

Search for every `async` function and `Promise` chain that does not have error handling:

```bash
# Find async functions
grep -rn --include="*.ts" --include="*.tsx" -E "async\s+function|async\s+\(" src/

# Find .then() without .catch()
grep -rn --include="*.ts" --include="*.tsx" -E "\.then\(" src/ | grep -v "\.catch\("
```

For each async CDF call, the pattern must be:

```ts
// GOOD — errors are caught and surfaced
async function fetchAssets(sdk: CogniteClient) {
  try {
    const result = await sdk.assets.list({ limit: 100 });
    return result.items;
  } catch (error) {
    console.error("Failed to fetch assets:", error);
    throw error; // re-throw so the caller / query layer can handle it
  }
}
```

When using TanStack Query (`useQuery`/`useMutation`), the library catches errors automatically — but verify that `isError` and `error` are consumed in the component:

```tsx
const { data, isLoading, isError, error } = useQuery({
  queryKey: ["assets"],
  queryFn: () => fetchAssets(sdk),
});

if (isError) return <ErrorMessage error={error} />;  // must be present
```

---

## Step 4 — Ensure loading and error states are shown in every component

For each component that fetches data, verify it has three distinct UI states:

| State | Required UI |
|-------|-------------|
| Loading | Spinner, skeleton, or loading indicator |
| Error | User-readable message (not a raw error object or blank space) |
| Empty | "No results" / "Nothing here yet" message (not a blank list) |

Search for components that render data without checking loading state:

```bash
grep -rn --include="*.tsx" -E "\.(map|filter|find)\(" src/ | grep -v "isLoading\|isPending\|skeleton\|Skeleton"
```

For each hit, read the component to confirm loading/error/empty states are handled above the `.map()` call.

---

## Step 5 — Narrow types before use

External data (CDF responses, URL params, `localStorage`, `JSON.parse`) must be validated before use. TypeScript types alone are not runtime guarantees.

```bash
# Find JSON.parse without validation
grep -rn --include="*.ts" --include="*.tsx" -E "JSON\.parse\(" src/

# Find localStorage reads
grep -rn --include="*.ts" --include="*.tsx" -E "localStorage\.(get|set)Item" src/

# Find useSearchParams usage
grep -rn --include="*.ts" --include="*.tsx" -E "useSearchParams|searchParams\.get" src/
```

For each hit, verify the value is either:
- Validated with Zod: `const parsed = MySchema.safeParse(raw);`
- Guarded with a type guard: `if (typeof value !== "string") return;`
- Handled with a nullish fallback: `const id = params.get("id") ?? defaultId;`

Do not cast external data with `as MyType` — that bypasses runtime safety.

---

## Step 6 — Handle null, undefined, and empty arrays

Read every component that accesses properties of data returned from CDF or passed via props.

Common patterns to flag:

```tsx
// BAD — crashes if data is undefined
const name = asset.properties.space.Asset.name;

// GOOD — optional chaining
const name = asset.properties?.["my-space"]?.["Asset"]?.name ?? "Unknown";

// BAD — crashes if items is undefined
items.map(renderItem);

// GOOD
(items ?? []).map(renderItem);

// BAD — crashes if the array is empty
const first = items[0].name;

// GOOD
const first = items.at(0)?.name ?? "—";
```

Search for direct array index access:
```bash
grep -rn --include="*.tsx" --include="*.ts" -E "\w+\[0\]\." src/
```

---

## Step 7 — Verify useEffect cleanup

Every `useEffect` that sets up a subscription, timer, event listener, or async operation that can outlive the component must return a cleanup function.

```bash
grep -rn --include="*.tsx" --include="*.ts" -B 2 -A 15 "useEffect" src/
```

For each `useEffect`, check:

| Pattern | Required cleanup |
|---------|-----------------|
| `addEventListener` | `removeEventListener` |
| `setInterval` / `setTimeout` | `clearInterval` / `clearTimeout` |
| CDF streaming / SSE | Close the stream |
| `fetch` / CDF SDK call | `AbortController.abort()` |
| Zustand / event emitter subscription | `unsubscribe()` |

Missing cleanup causes memory leaks and stale state updates on unmounted components.

```ts
// GOOD — async fetch with abort
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

## Step 8 — Check edge cases

For each feature, verify behaviour for:

- **Empty data**: zero-item lists, zero-result CDF queries
- **Single item**: list with exactly one entry (common source of off-by-one rendering bugs)
- **Maximum data**: what happens when CDF returns the full `limit` and there are more pages? Is pagination communicated?
- **Concurrent requests**: if the user triggers a new search before the previous completes, is the stale result discarded?
- **Network offline**: does the app show a meaningful message or silently fail?

For Atlas tool `execute` functions, verify every `args` field is present and within expected bounds before calling CDF:

```ts
execute: async (args) => {
  if (!args.assetId || typeof args.assetId !== "string") {
    return { output: "Missing or invalid assetId", details: null };
  }
  // ... safe to proceed
}
```

---

## Step 9 — Verify 429 / rate-limit backoff with jitter

When CDF or any external API returns **429 Too Many Requests** (or similar rate-limit signals), the app must back off gracefully — not hammer the service with immediate retries.

### Search for existing retry logic

```bash
# Find retry/backoff patterns
grep -rn --include="*.ts" --include="*.tsx" -i -E "(retry|backoff|retryAfter|retry.after|429|rate.limit|throttle)" src/

# Find raw fetch/axios interceptors that might handle retries
grep -rn --include="*.ts" --include="*.tsx" -E "interceptors\.(response|request)\.use" src/
```

### What to check

For each CDF SDK call site or API call:

| Check | Requirement |
|-------|-------------|
| Backoff strategy | **Exponential backoff with jitter** — not fixed-interval retries |
| `Retry-After` header | Respected when present in the 429 response |
| Retry cap | Bounded maximum attempts (e.g. 3–5 retries) — never infinite |
| User feedback | UI shows a loading/degraded state during retries — not silent |
| Synchronized retries | Jitter prevents multiple components from retrying at the exact same moment (thundering herd) |

### Reference implementation

If no retry logic exists for CDF calls, recommend this pattern:

```typescript
async function fetchWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error: unknown) {
      if (
        attempt === maxRetries ||
        !(error instanceof Error) ||
        !('status' in error && (error as any).status === 429)
      ) {
        throw error;
      }

      // Exponential backoff with jitter
      const baseDelay = Math.pow(2, attempt) * 1000;
      const jitter = Math.random() * 1000;
      await new Promise((resolve) => setTimeout(resolve, baseDelay + jitter));
    }
  }
  throw new Error('Unreachable');
}
```

### Severity guide

| Pattern found | Severity |
|--------------|----------|
| No retry logic at all for CDF calls | MEDIUM — add backoff wrapper |
| Fixed-interval retries (e.g. retry every 1s) | MEDIUM — add exponential backoff + jitter |
| Immediate tight retries on 429 | HIGH — thundering herd risk |
| Infinite or unbounded retries | HIGH — can overwhelm shared quotas |
| Backoff exists but no jitter | LOW — add jitter to prevent synchronized retries |

---

## Step 10 — Report findings

Produce a structured report:

| Severity | File | Line | Issue | Recommendation |
|----------|------|------|-------|----------------|
| HIGH | `src/hooks/useAssets.ts` | 34 | Unhandled promise rejection in fetchAssets | Wrap in try/catch and surface error state |
| MEDIUM | `src/components/AssetList.tsx` | 12 | No empty state rendered when `items.length === 0` | Add empty state message |
| LOW | `src/App.tsx` | — | No top-level ErrorBoundary | Wrap `<App>` content with ErrorBoundary |

If no issues are found in a step, state "No issues found" for that step. Do not skip steps silently.

---

## Done

Summarize findings by severity. Flag any HIGH issues that could cause data loss, crashes in production, or misleading UI states, and list them first for immediate attention.
