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

> Do **not** install `vite-plugin-node-polyfills`. It introduces a different set of
> transitive-dep conflicts. Use explicit `process`, `util`, `assert` package aliases instead.

---

## Step 2 — Vite config

Read [vite-config.md](references/vite-config.md) for the complete `vite.config.ts`. Apply it verbatim.

Key points:

- **`resolve.dedupe` includes `react`, `react-dom`, `react/jsx-runtime`, `three`** — pnpm symlinks can create separate module instances in a monorepo; `dedupe` forces one copy
- **Manual `util/`, `assert/`, `process/browser` aliases** — not a plugin. These handle the top-level imports. The `process`, `util`, `assert` npm packages must be in `dependencies` (Step 1)
- **`optimizeDeps.include` lists `process`, `util`, `assert`, `three`, `@cognite/reveal`** — pre-bundles them so esbuild handles CJS→ESM; Vite cannot auto-discover bare polyfill imports
- **`worker.format: 'es'`** — Reveal spawns ES module web workers; without this they fail silently
- **Never put `@cognite/dune-industrial-components` in `optimizeDeps.exclude`** — forces raw ESM, re-introduces React duplication even with dedupe

---

## Step 3 — main.tsx

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

Keep `DuneAuthProvider` from `@cognite/dune` as the auth provider — **not** `CDFAuthenticationProvider`.

---

## Step 4 — Provider placement (critical)

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

## Step 5 — Implementation

**Decide the pattern first — before reading any code.**

Use **Pattern B (model browser)** unless you can answer YES to all three of these:
1. The app already has a `DMInstanceRef` in scope — passed in as a prop or route param, not something to be fetched
2. The user has confirmed that instance has `CogniteVisualizable.object3D → CogniteCADNode` linkage in their CDF data model
3. The user explicitly asked for FDM-linked 3D, not just "show a 3D viewer"

If any answer is NO or uncertain — use Pattern B. It works with every CDF project that has 3D models, requires zero FDM setup, and is much easier to debug. Pattern A silently renders nothing when FDM linkage is missing.

**Pattern B:** Read the "Pattern B (default)" section of [references/implementation.md](references/implementation.md).

**Pattern A (only if gate above passed):** Read the "Pattern A (fallback)" section of [references/implementation.md](references/implementation.md).

**Two files to create:** `src/components/ViewerContent.tsx` (canvas only, no providers) and `src/App.tsx` (owns all providers + model selection logic).

**Critical rules that both patterns share:**

- `ViewerContent` must contain **no providers** — `CacheProvider`, `RevealKeepAlive`, and `RevealProvider` all live in `App.tsx` (see Step 5 for why)
- `resources` prop for `Reveal3DResources` must be `useMemo`'d; `onModelsLoaded` must be `useCallback`'d — inline values cause infinite model reload loops
- `onSelect`/`onLoad` callbacks passed into child components must be `useCallback`'d at the call site, and called from `useEffect` inside the child — not during render
- `sdk` passed to `RevealProvider` must be `useMemo`'d keyed on `client.project`
- Lazy-load `ViewerContent` with `React.lazy` + `Suspense` to avoid blocking the initial bundle

---

## Step 6 — Container height

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
