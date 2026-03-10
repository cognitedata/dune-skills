---
name: reveal-3d
description: "MUST be used whenever adding a 3D viewer to a Dune app using Cognite Reveal. Do NOT manually wire up RevealProvider, RevealCanvas, or FDM model/styling hooks — this skill handles installation, component structure, and hook wiring. Triggers: 3D viewer, reveal, CAD model, 3D visualization, RevealProvider, RevealCanvas, useFocusCamera, useInstanceStyling, FDM 3D, asset 3D model."
allowed-tools: Read, Glob, Grep, Edit, Write, Bash
metadata:
  argument-hint: "[FDM instance variable name or description, e.g. 'asset' or 'selectedEquipment']"
---

# Reveal 3D Viewer

Add a 3D CAD model viewer to this Dune app using Cognite Reveal.

FDM instance to visualize: **$ARGUMENTS**

## Your job

Complete these steps in order. Read each file before modifying it.

---

## Step 1 — Install dependencies

```bash
npm install @cognite/reveal @tanstack/react-query three dune-fe-auth
```

> `@cognite/dune-industrial-components` must already be installed. The `reveal` entry point is available at `@cognite/dune-industrial-components/reveal`.

---

## Step 2 — Understand the module

The `reveal` module exports:

**Components:**
- `RevealProvider` — wraps children with a `Cognite3DViewer` context. Requires `sdk` (CogniteClient) and optionally `color` (THREE.Color) and `viewerOptions`.
- `RevealCanvas` — renders the viewer's DOM element and portals children into it.
- `Reveal3DResources` — loads CAD models and applies `instanceStyling`.

**Key hooks:**
- `useModelsForInstanceQuery(instance)` — fetches `TaggedAddResourceOptions[]` for an FDM instance via Core DM
- `useFocusCamera(instance)` — returns a callback that focuses the camera on the instance's bounding box
- `useInstanceStyling(instance)` — returns `InstanceStylingGroup[]` highlighting the instance and its related 3D nodes
- `useReveal()` — access the raw `Cognite3DViewer` instance

**Types:**
- `DMInstanceRef` (from `@cognite/reveal`) — `{ space: string; externalId: string }`
- `AddCadResourceOptions` — `{ modelId: number; revisionId: number; styling?: ... }`
- `ViewerOptions` — subset of `Cognite3DViewerOptions`

---

## Step 3 — Create the inner viewer component

This component must be rendered **inside** `<RevealProvider>` and `<RevealCanvas>`.

```tsx
import { useCallback, useEffect, useMemo, useState } from 'react';
import * as THREE from 'three';
import type { DMInstanceRef } from '@cognite/reveal';
import {
  RevealProvider,
  RevealCanvas,
  Reveal3DResources,
  useModelsForInstanceQuery,
  useReveal,
  useFocusCamera,
  useInstanceStyling,
  type AddCadResourceOptions,
  type TaggedAddResourceOptions,
  type ViewerOptions,
} from '@cognite/dune-industrial-components/reveal';
import { useCDF } from 'dune-fe-auth';

const CAMERA_FOCUS_DELAY_MS = 100;
const BACKGROUND_COLOR = new THREE.Color(0x404040);

const defaultViewerOptions: ViewerOptions = {
  loadingIndicatorStyle: { placement: 'topRight', opacity: 0.1 },
  antiAliasingHint: 'msaa2+fxaa',
  ssaoQualityHint: 'medium',
};

function getFirstCadModel(
  models: TaggedAddResourceOptions[]
): AddCadResourceOptions | undefined {
  const model = models[0];
  if (!model || model.type !== 'cad') return undefined;
  return { ...model.addOptions, styling: { default: { renderGhosted: true } } };
}

interface Viewer3DInternalProps {
  instance: DMInstanceRef;
}

function Viewer3DInternal({ instance }: Viewer3DInternalProps) {
  const viewer = useReveal();
  const { data: models, isLoading } = useModelsForInstanceQuery(instance);
  const [modelsLoaded, setModelsLoaded] = useState(false);

  const selectedModel = useMemo(() => getFirstCadModel(models ?? []), [models]);
  const resources = useMemo(
    () => (selectedModel ? [selectedModel] : []),
    [selectedModel]
  );

  const focusCamera = useFocusCamera(instance);
  const instanceStyling = useInstanceStyling(instance);

  useEffect(() => {
    viewer.setBackgroundColor({ alpha: 0 });
  }, [viewer]);

  useEffect(() => {
    if (!modelsLoaded) return;
    const timer = setTimeout(focusCamera, CAMERA_FOCUS_DELAY_MS);
    return () => clearTimeout(timer);
  }, [modelsLoaded, focusCamera]);

  const handleModelsLoaded = useCallback(() => setModelsLoaded(true), []);

  if (isLoading) return <div>Loading 3D model...</div>;
  if (resources.length === 0) return <div>No 3D data available</div>;

  return (
    <RevealCanvas>
      <Reveal3DResources
        resources={resources}
        instanceStyling={instanceStyling}
        onModelsLoaded={handleModelsLoaded}
      />
    </RevealCanvas>
  );
}
```

---

## Step 4 — Wrap with RevealProvider

`RevealProvider` initializes the viewer. Memoize the SDK to prevent re-initialization on every render.

```tsx
interface Viewer3DProps {
  instance: DMInstanceRef;
}

export function Viewer3D({ instance }: Viewer3DProps) {
  const { sdk: client } = useCDF();
  // Memoize to prevent RevealProvider from recreating the viewer on every render
  const sdk = useMemo(() => client, [client?.project]);

  if (!sdk) return null;

  return (
    <RevealProvider sdk={sdk} color={BACKGROUND_COLOR} viewerOptions={defaultViewerOptions}>
      <Viewer3DInternal instance={instance} />
    </RevealProvider>
  );
}
```

---

## Step 5 — Integrate into your app

1. Find the component or page that displays information about the FDM instance (e.g. an asset detail panel).
2. Import `Viewer3D` and render it with the instance reference:

```tsx
<div style={{ width: '100%', height: 400 }}>
  <Viewer3D instance={{ space: instance.space, externalId: instance.externalId }} />
</div>
```

3. The viewer needs an explicit height. Use a container with `height` set — otherwise the canvas will be 0px tall.

---

## Step 6 — Verify

- The 3D viewer renders and loads the CAD model
- The camera auto-focuses on the selected instance
- The instance is highlighted in the model

If no model loads, verify that the FDM instance has a `CogniteVisualizable.object3D` relation connecting it to a `CogniteCADNode`.
