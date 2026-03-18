---
name: reveal-3d
description: "Sets up a 3D CAD model viewer in a Dune app using Cognite Reveal via @cognite/dune-industrial-components/reveal. Use this skill whenever the user mentions 3D viewer, 3D visualization, reveal, CAD model, RevealProvider, RevealCanvas, Reveal3DResources, FDM 3D mapping, asset 3D model, loading a 3D model, or wants to display any Cognite 3D content in a Dune application — even if they don't explicitly say 'Reveal' or '3D viewer'. Do NOT manually wire up RevealProvider, RevealCanvas, or model-loading hooks without consulting this skill first."
metadata:
  argument-hint: "[FDM instance variable name or description, e.g. 'asset' or 'selectedEquipment']"
---

# Reveal 3D Viewer

Add a Cognite Reveal 3D viewer to a Dune app. The viewer renders CAD models from CDF and optionally highlights FDM-linked assets.

FDM instance to visualize: **$ARGUMENTS**

---

## Step 1 — Install dependencies

```bash
npm install @cognite/reveal three process
npm install "@cognite/dune-industrial-components@github:cognitedata/dune-industrial-components#semver:*"
```

For local dev: `"@cognite/dune-industrial-components": "file:../dune-industrial-components"`

**All four packages** (`@cognite/reveal`, `three`, `process`, `@cognite/dune-industrial-components`) **must** appear in `package.json` dependencies. Verify after install.

The app must already have `react`, `react-dom`, `@cognite/sdk`, `@tanstack/react-query`, and `dune-fe-auth`.

---

## Step 2 — Configure Vite for Reveal

`@cognite/reveal` bundles Node.js modules (`util`, `assert`) that reference `process` — which doesn't exist in the browser. Without polyfills, the app crashes with `ReferenceError: process is not defined`. Reveal also has its own Three.js copy; without deduplication you get `WARNING: Multiple instances of Three.js` and broken rendering.

Read [vite-config.md](vite-config.md) and apply both changes:

1. **`main.tsx`** — add `process` polyfill as the very first import (before React)
2. **`vite.config.ts`** — add `define`, `resolve.alias`, `resolve.dedupe`, and `optimizeDeps.include`

---

## Step 3 — Component architecture

The nesting order matters — each component depends on context from its parent:

```
<RevealKeepAlive>           ← Survives StrictMode + navigation remounts
  <RevealProvider sdk={..}> ← Creates Cognite3DViewer, provides context
    <RevealCanvas>          ← Mounts WebGL canvas into DOM
      <Reveal3DResources /> ← Loads models, applies styling
    </RevealCanvas>
  </RevealProvider>
</RevealKeepAlive>
```

**Why RevealKeepAlive?** Creating a `Cognite3DViewer` is expensive (~2-3s). Without KeepAlive, navigating away and back destroys and recreates it. KeepAlive stores the viewer in a ref that survives child unmounts. It also prevents React StrictMode's mount→unmount→remount from disposing the viewer mid-lifecycle.

**Why memoize props?** `Reveal3DResources` uses `resources` and `onModelsLoaded` as effect dependencies. Inline arrays/callbacks create new references every render, causing the effect to re-run (unload models → reload models). Always `useMemo` for `resources` and `useCallback` for `onModelsLoaded`.

---

## Step 4 — Implementation

Two patterns depending on whether you're discovering models via FDM or loading directly by ID.

### Pattern A: FDM instance → auto-discover models

Use when you have a `DMInstanceRef` (space + externalId) and the instance is linked to 3D via Core DM (`CogniteVisualizable.object3D` → `CogniteCADNode`).

```tsx
import { useCallback, useMemo, useState } from 'react';
import * as THREE from 'three';
import type { DMInstanceRef } from '@cognite/reveal';
import {
  RevealProvider, RevealCanvas, Reveal3DResources, RevealKeepAlive,
  useModelsForInstanceQuery,
  type AddCadResourceOptions, type TaggedAddResourceOptions, type ViewerOptions,
} from '@cognite/dune-industrial-components/reveal';
import { useCDF } from 'dune-fe-auth';

const BG = new THREE.Color(0x1a1a2e);
const OPTS: ViewerOptions = {
  loadingIndicatorStyle: { placement: 'topRight', opacity: 0.1 },
  antiAliasingHint: 'msaa2+fxaa',
  ssaoQualityHint: 'medium',
};

function pickFirstCad(models: TaggedAddResourceOptions[]): AddCadResourceOptions | undefined {
  const m = models[0];
  return m?.type === 'cad' ? { ...m.addOptions, styling: { default: { renderGhosted: true } } } : undefined;
}

function ViewerInternal({ instance }: { instance: DMInstanceRef }) {
  const { data: models, isLoading } = useModelsForInstanceQuery(instance);
  const [loaded, setLoaded] = useState(false);
  const selected = useMemo(() => pickFirstCad(models ?? []), [models]);
  const resources = useMemo(() => (selected ? [selected] : []), [selected]);
  const onLoaded = useCallback(() => setLoaded(true), []);

  if (isLoading) return <div>Loading 3D model...</div>;
  if (!resources.length) return <div>No 3D data available</div>;

  return (
    <RevealCanvas>
      <Reveal3DResources resources={resources} onModelsLoaded={onLoaded} />
    </RevealCanvas>
  );
}

export function Viewer3D({ instance }: { instance: DMInstanceRef }) {
  const { sdk: client } = useCDF();
  const sdk = useMemo(() => client, [client?.project]);
  if (!sdk) return null;

  return (
    <RevealKeepAlive>
      <RevealProvider sdk={sdk} color={BG} viewerOptions={OPTS}>
        <ViewerInternal instance={instance} />
      </RevealProvider>
    </RevealKeepAlive>
  );
}
```

### Pattern B: Direct model/revision IDs

Use when you already know `modelId` and `revisionId` (e.g., from a model browser or config).

```tsx
export function DirectViewer({ modelId, revisionId }: { modelId: number; revisionId: number }) {
  const { sdk: client } = useCDF();
  const sdk = useMemo(() => client, [client?.project]);
  const resources = useMemo(() => [{ modelId, revisionId }], [modelId, revisionId]);
  const onLoaded = useCallback(() => {}, []);
  if (!sdk) return null;

  return (
    <RevealKeepAlive>
      <RevealProvider sdk={sdk} color={BG} viewerOptions={OPTS}>
        <RevealCanvas>
          <Reveal3DResources resources={resources} onModelsLoaded={onLoaded} />
        </RevealCanvas>
      </RevealProvider>
    </RevealKeepAlive>
  );
}
```

---

## Step 5 — Integrate into your page

The viewer canvas fills its container via `width: 100%; height: 100%`. The container **must have an explicit height** — otherwise the canvas collapses to 0px.

```tsx
<div style={{ width: '100%', height: '70vh' }}>
  <Viewer3D instance={{ space: 'my-space', externalId: 'my-asset' }} />
</div>
```

For heavy components like Reveal, lazy-load to avoid blocking the initial bundle:

```tsx
const RevealViewer = lazy(() => import('./RevealViewer').then(m => ({ default: m.RevealViewer })));
```

---

## Step 6 — Verify

Check the browser console for these common issues:

| Symptom | Cause | Fix |
|---------|-------|-----|
| `process is not defined` | Missing Vite polyfills | Apply [vite-config.md](vite-config.md) |
| `Multiple instances of Three.js` | No `resolve.alias` for `three` | Point alias to `node_modules/three/build/three.module.js` |
| `ObjectUnsubscribedError` | Viewer disposed by StrictMode | Wrap in `<RevealKeepAlive>` |
| Black screen, "Loaded" badge | Props not memoized → infinite reload | `useMemo` for `resources`, `useCallback` for callbacks |
| Canvas is 0px tall | Container has no height | Set explicit `height` on parent div |
| No model found | FDM instance not linked to 3D | Check `CogniteVisualizable.object3D` → `CogniteCADNode` relation |

---

## API reference

See the full exports from `@cognite/dune-industrial-components/reveal`:

**Components:** `RevealProvider`, `RevealCanvas`, `Reveal3DResources`, `RevealKeepAlive`, `CacheProvider`

**Hooks:** `useReveal`, `useModelsForInstanceQuery`, `use3dModels`, `useFdmAssetMappings`, `usePrefetchedFdmMappings`, `useRemoveNonReferencedModels`, `use3dRelatedEdgeConnections`, `use3dRelatedDirectConnections`, `use3dDataForSelectedInstance`, `useInstancesWithBoundingBoxes`

**Types:** `AddCadResourceOptions`, `TaggedAddResourceOptions`, `ViewerOptions`, `RevealContextProps`, `InstanceStylingGroup`, `FdmAssetStylingGroup`, `CadModelOptions`, `RevealGeometryFilter`, `CogniteModel`
