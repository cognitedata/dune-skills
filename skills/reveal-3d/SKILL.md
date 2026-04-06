---
name: reveal-3d
description: "Sets up a 3D CAD model viewer in a Dune app using Cognite Reveal via @cognite/dune-industrial-components/reveal. Use this skill whenever the user mentions 3D viewer, 3D visualization, reveal, CAD model, RevealProvider, RevealCanvas, Reveal3DResources, FDM 3D mapping, asset 3D model, loading a 3D model, or wants to display any Cognite 3D content in a Dune application — even if they don't explicitly say 'Reveal' or '3D viewer'. Do NOT manually wire up RevealProvider, RevealCanvas, or model-loading hooks without consulting this skill first."
metadata:
  argument-hint: "[FDM instance variable name or description, e.g. 'asset' or 'selectedEquipment']"
---

# Reveal 3D Viewer

Add a Cognite Reveal 3D viewer to a Dune app. Renders CAD models from CDF, with support for FDM-linked assets or direct model/revision IDs.

FDM instance to visualize: **$ARGUMENTS**

---

## Before you start

Read these files before touching anything:

- `package.json` — note `react`/`react-dom` versions and existing deps
- `vite.config.ts` — you will **replace** it entirely (new Dune apps have a standalone config, not a shared base config)
- `src/main.tsx` — you will prepend two lines to it

---

## Step 1 — Install packages

```bash
pnpm add @cognite/reveal three process util assert ajv
pnpm add "@cognite/dune-industrial-components@github:cognitedata/dune-industrial-components#semver:*"
pnpm add -D @types/three
pnpm install --no-frozen-lockfile
```

> **If running inside Cursor (sandbox):** the GitHub package install requires `git init`,
> which the Cursor sandbox blocks. If you see `git init ... Operation not permitted`, the
> install must be run with full permissions. In a Shell tool call, pass
> `required_permissions: ["all"]`.

**After install — check `three` version matches what `@cognite/reveal` requires:**

```bash
node -e "const r=require('./node_modules/@cognite/reveal/package.json'); console.log(r.peerDependencies?.three)"
node -e "console.log(require('./node_modules/three/package.json').version)"
```

If the installed `three` version is lower than `@cognite/reveal`'s peer requirement, update it:

```bash
pnpm add three@^<required-version>    # e.g. three@^0.180.0
pnpm install --no-frozen-lockfile
```

**Verify** `package.json` now has all of: `@cognite/reveal`, `three` (at the right version),
`process`, `util`, `assert`, `ajv`, `@cognite/dune-industrial-components`.

> **Why `ajv`?** `@cognite/dune-industrial-components` requires `ajv@>=8`. The monorepo
> root has `ajv@6` as a transitive dep. Without a direct `ajv@^8` in the app, pnpm
> picks up the root's v6 and you get a peer warning that can cause schema validation failures.

> Do **not** install `dune-fe-auth` as a real package. Its bundled code calls
> `React.__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED.ReactCurrentDispatcher`,
> which was removed in React 19. Dune monorepos use React 19 — loading the real
> `dune-fe-auth` crashes on every bundling strategy. Instead, create a shim (Step 2).

> Do **not** install `vite-plugin-node-polyfills`. It introduces a different set of
> transitive-dep conflicts. Use explicit `process`, `util`, `assert` package aliases instead.

---

## Step 2 — Create the dune-fe-auth shim

`@cognite/dune-industrial-components/reveal` calls `useCDF()` from `dune-fe-auth` internally.
Create a one-line shim and alias Vite to serve it in place of the real package:

```ts
// src/dune-fe-auth-shim.ts
// useDune and useCDF have identical signatures: { sdk, isLoading, error }
export { useDune as useCDF } from '@cognite/dune';
```

This prevents `dune-fe-auth`'s React 18 code from ever loading. The alias is added in Step 3.

---

## Step 3 — Vite config

Read [vite-config.md](vite-config.md) for the complete `vite.config.ts`. Apply it verbatim.

Key points:

- **`dune-fe-auth` alias → shim** — intercepts every internal import, serves `useDune` as `useCDF`, no version conflict
- **`resolve.dedupe` includes `react`, `react-dom`, `react/jsx-runtime`, `three`** — pnpm symlinks can create separate module instances in a monorepo; `dedupe` forces one copy
- **Manual `util/`, `assert/`, `process/browser` aliases** — not a plugin. These handle the top-level imports. The `process`, `util`, `assert` npm packages must be in `dependencies` (Step 1)
- **`optimizeDeps.include` lists `process`, `util`, `assert`, `three`, `@cognite/reveal`** — pre-bundles them so esbuild handles CJS→ESM; Vite cannot auto-discover bare polyfill imports
- **`worker.format: 'es'`** — Reveal spawns ES module web workers; without this they fail silently
- **Never put `@cognite/dune-industrial-components` or `dune-fe-auth` in `optimizeDeps.exclude`** — forces raw ESM, re-introduces React duplication even with dedupe

---

## Step 4 — main.tsx

Add the `process` polyfill as the **very first two lines** — before any import, before React:

```tsx
import process from 'process';
(window as unknown as Record<string, unknown>).process = process;

// all other imports below ↓
import { DuneAuthProvider } from '@cognite/dune';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App.tsx';
import './styles.css';

const queryClient = new QueryClient({
  defaultOptions: { queries: { staleTime: 5 * 60 * 1000, gcTime: 10 * 60 * 1000 } },
});

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <DuneAuthProvider>
        <App />
      </DuneAuthProvider>
    </QueryClientProvider>
  </React.StrictMode>
);
```

Keep `DuneAuthProvider` from `@cognite/dune` as the auth provider — **not** `CDFAuthenticationProvider` from `dune-fe-auth`.

---

## Step 5 — Provider placement (critical)

**Getting this wrong causes `ObjectUnsubscribedError: object unsubscribed` at model load time.**

`CacheProvider` and `RevealKeepAlive` must be **always mounted at page/app level**. `RevealProvider` is conditional (only renders when a model is selected).

```
App (always rendered)
  CacheProvider            ← always mounted
    RevealKeepAlive        ← always mounted
      <sidebar>            ← model picker lives here
      {selected && (
        RevealProvider     ← conditional, only when model is ready
          <RevealCanvas>
            <Reveal3DResources />
      )}
```

**Why:** React StrictMode double-invokes effects on every component at first mount.
If `RevealKeepAlive` is inside the same conditionally-rendered component as `RevealProvider`,
StrictMode fires *both* cleanup cycles together — `RevealKeepAlive` disposes the
viewer while `RevealProvider`'s async model-loading effect is still in-flight.

When `RevealKeepAlive` is at App level, its StrictMode cycle completes at startup
with no viewer yet (nothing to dispose). By the time `RevealProvider` conditionally
mounts, `RevealKeepAlive`'s `viewerRef` is stable — and `RevealProvider` skips
viewer disposal when keepAlive context is present.

**Pattern that breaks:**
```
{selected && (
  <MyViewerComponent>      ← ❌ RevealKeepAlive co-located with RevealProvider
    <CacheProvider>
      <RevealKeepAlive>
        <RevealProvider>
```

---

## Step 6 — Implementation

**Default pattern: Pattern B (model browser — auto-loads models).**
Uses `sdk.models3D.list()` to auto-discover all 3D models in the CDF project and
`sdk.revisions3D.list()` to pick the best revision automatically. No manual IDs or
FDM linkage needed — works with any CDF project that has 3D models. Always start here.

Use **Pattern A (FDM)** only when the app receives a `DMInstanceRef` and the instance
has `CogniteVisualizable.object3D → CogniteCADNode` linkage.
Full Pattern A code is in [references/implementation.md](references/implementation.md).

Two files to create: a canvas-only content component, and the App that owns all providers.

### `src/components/ViewerContent.tsx` — canvas only, no providers (Pattern B)

```tsx
import { useCallback, useMemo } from 'react';
import {
  Reveal3DResources,
  RevealCanvas,
  type AddCadResourceOptions,
} from '@cognite/dune-industrial-components/reveal';

export interface ViewerContentProps {
  modelId: number;
  revisionId: number;
}

/**
 * Canvas-only — no CacheProvider, RevealKeepAlive, or RevealProvider here.
 * All providers live in App.tsx so React StrictMode double-invoke completes
 * at startup before any model loading starts.
 */
export function ViewerContent({ modelId, revisionId }: ViewerContentProps) {
  // AddCadResourceOptions is just { modelId, revisionId } — no `type` field.
  // Do NOT use { type: 'cad', modelId, revisionId } — that is TaggedAddResourceOptions.
  const resources = useMemo<AddCadResourceOptions[]>(
    () => [{ modelId, revisionId }],
    [modelId, revisionId]
  );
  const onLoaded = useCallback(() => {}, []);

  return (
    <RevealCanvas>
      <Reveal3DResources resources={resources} onModelsLoaded={onLoaded} />
    </RevealCanvas>
  );
}
```

### `src/App.tsx` — owns all providers + model browser (Pattern B)

```tsx
import { lazy, Suspense, useCallback, useEffect, useMemo, useState } from 'react';
import * as THREE from 'three';
import { useDune } from '@cognite/dune';
import { useInfiniteQuery, useQuery } from '@tanstack/react-query';
import type { Model3D, Revision3D } from '@cognite/sdk';
import {
  CacheProvider,
  RevealKeepAlive,
  RevealProvider,
  type ViewerOptions,
} from '@cognite/dune-industrial-components/reveal';

const ViewerContent = lazy(() =>
  import('./components/ViewerContent').then((m) => ({ default: m.ViewerContent }))
);

const BG = new THREE.Color(0x1a1a2e);
const VIEWER_OPTIONS: ViewerOptions = {
  loadingIndicatorStyle: { placement: 'topRight', opacity: 0.1 },
  antiAliasingHint: 'msaa2+fxaa',
  ssaoQualityHint: 'medium',
};

type SelectedModel = { modelId: number; revisionId: number };

function useModels(query?: string) {
  const { sdk } = useDune();
  return useInfiniteQuery({
    queryKey: ['3d-models', query],
    queryFn: ({ pageParam }: { pageParam?: string }) =>
      sdk.models3D.list({ limit: 1000, cursor: pageParam }) as Promise<{
        items: Model3D[];
        nextCursor?: string;
      }>,
    initialPageParam: undefined as string | undefined,
    getNextPageParam: (page) => page.nextCursor,
    enabled: !!sdk,
    select: useCallback(
      (data: any) => ({
        ...data,
        pages: data.pages.map((p: any) => ({
          ...p,
          items: p.items
            .map((m: Model3D) => ({ ...m, name: m.name.trim() }))
            .filter((m: Model3D) =>
              query ? m.name.toLowerCase().includes(query.toLowerCase()) : true
            ),
        })),
      }),
      [query]
    ),
  });
}

function useBestRevision(modelId?: number) {
  const { sdk } = useDune();
  return useQuery({
    queryKey: ['3d-revisions', modelId],
    queryFn: async () => {
      if (!modelId) return null;
      const all: Revision3D[] = await sdk.revisions3D
        .list(modelId)
        .autoPagingToArray({ limit: -1 });
      const published = all.filter((r) => r.published);
      const candidates = published.length ? published : all;
      return candidates.reduce((best, cur) =>
        best.createdTime > cur.createdTime ? best : cur
      ) ?? null;
    },
    enabled: !!sdk && !!modelId,
  });
}

// RULE 1: onSelect MUST be wrapped in useCallback at the call site.
//   An inline arrow `(m) => setSelected(m)` creates a new reference every render.
//   The useEffect([..., onSelect]) below fires on every render → infinite loop.
//
// RULE 2: call onSelect from useEffect, NOT during render.
//   An if-block during render calling onSelect also causes infinite loops.

function ModelBrowser({ onSelect }: { onSelect: (m: SelectedModel) => void }) {
  const [query, setQuery] = useState('');
  const [pendingId, setPendingId] = useState<number>();
  const { data } = useModels(query);
  const { data: revision } = useBestRevision(pendingId);
  const models = data?.pages.flatMap((p) => p.items) ?? [];

  useEffect(() => {
    if (revision && pendingId) {
      onSelect({ modelId: pendingId, revisionId: revision.id });
    }
  }, [revision, pendingId, onSelect]);

  return (
    <div>
      <input
        placeholder="Search models…"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      {models.map((m) => (
        <button key={m.id} onClick={() => setPendingId(m.id)}>
          {m.name}
        </button>
      ))}
    </div>
  );
}

export default function App() {
  const { sdk: client, isLoading } = useDune();
  // Memoize on sdk.project so RevealProvider only remounts on project change
  const sdk = useMemo(() => client, [client.project]);
  const [selected, setSelected] = useState<SelectedModel | null>(null);

  // useCallback is mandatory — see ModelBrowser RULE 1 above
  const handleSelect = useCallback((m: SelectedModel) => {
    setSelected((prev) => (!prev || prev.modelId !== m.modelId ? m : prev));
  }, []);

  if (isLoading) return <div>Connecting to CDF…</div>;

  return (
    <CacheProvider>
      <RevealKeepAlive>
        <div style={{ display: 'flex', height: '100vh' }}>
          <aside style={{ width: 280, overflowY: 'auto' }}>
            <ModelBrowser onSelect={handleSelect} />
          </aside>
          <div style={{ flex: 1, position: 'relative' }}>
            {selected && (
              <RevealProvider sdk={sdk} color={BG} viewerOptions={VIEWER_OPTIONS}>
                <Suspense fallback={<div>Loading viewer…</div>}>
                  <ViewerContent
                    modelId={selected.modelId}
                    revisionId={selected.revisionId}
                  />
                </Suspense>
              </RevealProvider>
            )}
          </div>
        </div>
      </RevealKeepAlive>
    </CacheProvider>
  );
}
```

> **`onSelect` must be wrapped in `useCallback` at the call site** — an inline arrow
> `(m) => setSelected(m)` creates a new reference every render, which triggers the
> `useEffect([..., onSelect])` above on every render → infinite loop.

---

## Step 7 — Container height

`RevealCanvas` fills its container with `width: 100%; height: 100%`. The parent **must have an explicit height**:

```tsx
<div style={{ width: '100%', height: '70vh', position: 'relative' }}>
  <RevealProvider ...>
    <ViewerContent modelId={...} revisionId={...} />
  </RevealProvider>
</div>
```

---

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `git init ... Operation not permitted` during `pnpm install` | Cursor sandbox blocks git operations needed to clone the GitHub package | Re-run `pnpm install --no-frozen-lockfile` with `required_permissions: ["all"]` in the Shell tool call |
| `pnpm install` hangs for minutes with no output | Same GitHub package sandbox issue — pnpm is stuck waiting on a blocked syscall | Kill the process, re-run with `required_permissions: ["all"]` |
| `unmet peer three@0.180.0: found 0.177.0` (or similar version) | `@cognite/reveal` requires a specific `three` version; `pnpm add three` installs latest which may differ | Check `@cognite/reveal`'s peerDependencies, then `pnpm add three@^<required>` + reinstall |
| `unmet peer ajv@>=8: found 6.x` | Monorepo root has `ajv@6`; app needs `ajv@^8` for `@cognite/dune-industrial-components` | `pnpm add ajv` in the app (adds `^8`) |
| `Could not resolve "dune-fe-auth"` | Real `dune-fe-auth` not installed (correct) but alias missing | Add `'dune-fe-auth': path.resolve(__dirname, 'src/dune-fe-auth-shim.ts')` to `resolve.alias` |
| `ReactCurrentDispatcher` is undefined | Real `dune-fe-auth` loaded — its React 18 internal doesn't exist in React 19 | Do **not** install real `dune-fe-auth`; use the shim (Step 2) |
| `ReactCurrentDispatcher` even with shim | `@cognite/dune-industrial-components` or shim in `optimizeDeps.exclude` | Remove from `exclude`; let Vite pre-bundle them together |
| `ObjectUnsubscribedError: object unsubscribed` | `RevealKeepAlive` inside conditional component | Move `CacheProvider` + `RevealKeepAlive` to always-mounted App level |
| `Maximum update depth exceeded` | Inline `onLoad`/`onSelect` callback `(t) => setState(t)` re-creates every render | `useCallback((t) => setState(t), [])` at the call site; model browser must call `onSelect` from `useEffect`, not render phase |
| `Maximum update depth exceeded` (variant) | `onSelect`/`onReady` called during render in an `if`-block instead of a `useEffect` | Move the call into `useEffect([revision, pendingId, onSelect])` |
| `No QueryClient set` | `@tanstack/react-query` resolved to a different copy | Add `@tanstack/react-query` to `resolve.dedupe` and `optimizeDeps.include` |
| `process is not defined` at runtime | Missing runtime polyfill | First two lines of `main.tsx`: `import process from 'process'; window.process = process;` |
| `Could not resolve "inherits"` | Used `vite-plugin-node-polyfills` or wrong manual aliases | Remove the plugin; use package aliases (`util: 'util/'`, `assert: 'assert/'`) with those packages in `dependencies` |
| `Multiple instances of Three.js` | Two Three.js copies loaded | `resolve.alias.three` → `node_modules/three/build/three.module.js` and `three` in `resolve.dedupe` |
| Black screen / workers fail silently | Missing ES worker format | Add `worker: { format: 'es' }` to vite config |
| Canvas 0px tall | Container has no explicit height | `height: '70vh'` (or any fixed/flex height) on the parent div |
| No model found (FDM mode) | Instance not linked via Core DM (`CogniteVisualizable.object3D` → `CogniteCADNode`) | Use model browser (Pattern B) with `sdk.models3D.list()` as the default instead |

---

## API reference

**Page-level (always rendered):** `CacheProvider`, `RevealKeepAlive`

**Viewer-level (conditional):** `RevealProvider`, `RevealCanvas`, `Reveal3DResources`, `InstanceStylingProvider`

**Hooks:** `useModelsForInstanceQuery`, `use3dModels`, `useFdmAssetMappings`, `useReveal`, `useOptionalRevealKeepAlive`

**Types:** `AddCadResourceOptions`, `TaggedAddResourceOptions`, `ViewerOptions`, `DMInstanceRef` (from `@cognite/reveal`)

All exports from `@cognite/dune-industrial-components/reveal`.
