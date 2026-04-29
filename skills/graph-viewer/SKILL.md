---
name: graph-viewer
description: Integrates the lightweight CDF graph viewer component via useGraphViewer hook. Use when embedding a graph visualization, adding a knowledge graph, showing CDF data model relationships, displaying nodes and edges from Cognite Data Fusion, or extracting/componentizing a reusable hook from a larger codebase.
---

# Graph Viewer Lite

## Overview

`useGraphViewer` is a single hook that renders an interactive CDF graph. Give it a data model reference and an optional seed instance -- it returns a self-contained `<GraphCanvas>` component and controls.

```tsx
import { useGraphViewer } from "@skills/graph-viewer/code";
```

**Prerequisites**: The app must be wrapped in `@cognite/dune`'s `<DuneProvider>` for SDK access.

## Dependencies

The hook source lives in `@skills/graph-viewer/code/`. Add the following packages to your app's `package.json` before importing:

| Package         | Version    | Purpose                                              |
| --------------- | ---------- | ---------------------------------------------------- |
| `react`         | `^18.2.0`  | UI framework (peer)                                  |
| `@cognite/sdk`  | `^10.10.0` | CDF API client (instances, data models)              |
| `@cognite/dune` | `^2.1.0`   | Provides the authenticated SDK via `useDune()`       |
| `reagraph`      | `^4.30.8`  | WebGL graph rendering engine                         |
| `lucide-react`  | `^1.14.0`  | Icon set used by the node-type legend                |

Install with your package manager, e.g.:

```bash
npm install react@^18.2.0 @cognite/sdk@^10.10.0 @cognite/dune@^2.1.0 reagraph@^4.30.8 lucide-react@^1.14.0
```

## Quick Start

```tsx
import { useGraphViewer } from "@skills/graph-viewer/code";

function MyGraph() {
  const { GraphCanvas, isLoading, error } = useGraphViewer({
    dataModel: {
      space: "my-space",
      externalId: "my-data-model",
      version: "1",
    },
    instance: {
      space: "my-instance-space",
      externalId: "pump-001",
    },
  });

  if (isLoading) return <div>Loading graph...</div>;
  if (error) return <div>Error: {error}</div>;

  return <GraphCanvas className="h-[600px] w-full" />;
}
```

The canvas includes zoom controls and a node-type legend.

## Hook Config

```typescript
useGraphViewer(config: UseGraphViewerConfig): UseGraphViewerReturn
```

### `UseGraphViewerConfig`

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `dataModel.space` | `string` | Yes | CDF space of the data model |
| `dataModel.externalId` | `string` | Yes | External ID of the data model |
| `dataModel.version` | `string` | Yes | Version of the data model |
| `instance.space` | `string` | No | Space of the seed node to auto-load |
| `instance.externalId` | `string` | No | External ID of the seed node |
| `options` | `UseGraphViewerOptions` | No | Advanced configuration (see below) |

### `UseGraphViewerOptions`

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `maxNodes` | `number` | `1000` | Max nodes in the LRU buffer |
| `layout` | `LayoutType` | `"forceDirected2d"` | Initial layout algorithm |
| `whitelistedRelationProps` | `string[]` | all | Only follow these relation properties when expanding |
| `coreReverseQueries` | `[space, viewId, prop, isList][]` | `[]` | Reverse relations to query on expand |
| `viewPriorityConfig` | `ViewPriorityConfig` | defaults | Node type detection priority |
| `initialConnectionLimit` | `number` | `100` | Max connections fetched per expand |
| `visualConfig` | `Partial<GraphVisualConfig>` | defaults | Colors, icon size, palette |
| `themeConfig` | `Partial<GraphThemeConfig>` | defaults | Canvas background, label styling |
| `features` | `Partial<LiteFeatureFlags>` | all `true` | Toggle legend, zoom, expansion |

### `LiteFeatureFlags`

| Flag | Default | What it controls |
|------|---------|-----------------|
| `enableLegend` | `true` | Node-type color legend overlay |
| `enableZoomControls` | `true` | Zoom in/out/fit buttons |
| `enableNodeExpansion` | `true` | Double-click to expand neighbors |

## Return Value

### `UseGraphViewerReturn`

| Property | Type | Description |
|----------|------|-------------|
| `GraphCanvas` | `React.FC<{ className?: string }>` | Self-contained canvas -- just render it |
| `isLoading` | `boolean` | `true` while data model or seed is loading |
| `error` | `string \| null` | Error message, or `null` |
| `graphData` | `GraphData` | Current nodes, connections, nodeTypes |
| `stats` | `GraphStats \| null` | `{ totalNodes, totalConnections, nodeTypes, connectionTypes }` |
| `layout` | `LayoutType` | Current layout |
| `setLayout` | `(layout) => void` | Change layout algorithm |
| `selections` | `string[]` | Selected node/edge IDs |
| `setSelections` | `(ids) => void` | Programmatically select items |
| `selectedNode` | `GraphNode \| null` | Currently selected node |
| `selectedEdge` | `GraphEdge \| null` | Currently selected edge |
| `expandNode` | `(nodeId) => Promise<void>` | Expand a node's neighbors |
| `loadInstance` | `(space, externalId) => Promise<void>` | Load a new seed node (clears graph) |
| `fitView` | `() => void` | Fit all nodes into view |
| `zoomIn` | `() => void` | Zoom in |
| `zoomOut` | `() => void` | Zoom out |
| `clear` | `() => void` | Clear all nodes and edges |
| `graphRef` | `RefObject<GraphCanvasRef>` | Direct ref to Reagraph canvas |

### `LayoutType` values

`"forceDirected2d"` | `"forceDirected3d"` | `"treeTd2d"` | `"treeLr2d"` | `"radialOut2d"` | `"circular2d"`

## Examples

### Minimal embed

```tsx
function GraphPanel() {
  const { GraphCanvas } = useGraphViewer({
    dataModel: { space: "equipment", externalId: "EquipmentModel", version: "1" },
  });

  return <GraphCanvas className="h-full w-full" />;
}
```

No seed instance -- the canvas renders empty until you call `loadInstance`.

### With layout switcher and stats

```tsx
function GraphWithControls() {
  const { GraphCanvas, stats, layout, setLayout } = useGraphViewer({
    dataModel: { space: "equipment", externalId: "EquipmentModel", version: "1" },
    instance: { space: "assets", externalId: "pump-001" },
  });

  return (
    <div className="flex flex-col h-full">
      <div className="flex items-center gap-4 p-2 border-b">
        <select value={layout} onChange={(e) => setLayout(e.target.value as LayoutType)}>
          <option value="forceDirected2d">Force 2D</option>
          <option value="treeTd2d">Tree</option>
          <option value="radialOut2d">Radial</option>
          <option value="circular2d">Circular</option>
        </select>
        {stats && <span>{stats.totalNodes} nodes</span>}
      </div>
      <GraphCanvas className="flex-1" />
    </div>
  );
}
```

### Programmatic node loading

```tsx
function SearchAndGraph() {
  const { GraphCanvas, loadInstance, isLoading } = useGraphViewer({
    dataModel: { space: "equipment", externalId: "EquipmentModel", version: "1" },
  });

  const handleSearch = async (externalId: string) => {
    await loadInstance("assets", externalId);
  };

  return (
    <div className="flex flex-col h-full">
      <input
        placeholder="Enter node externalId..."
        onKeyDown={(e) => {
          if (e.key === "Enter") handleSearch(e.currentTarget.value);
        }}
      />
      {isLoading && <p>Loading...</p>}
      <GraphCanvas className="flex-1" />
    </div>
  );
}
```

### Disable features

```tsx
const { GraphCanvas } = useGraphViewer({
  dataModel: { space: "s", externalId: "dm", version: "1" },
  instance: { space: "s", externalId: "node-1" },
  options: {
    features: {
      enableLegend: false,
      enableZoomControls: false,
      enableNodeExpansion: false,
    },
  },
});
```

### Controlled expansion with whitelisted relations

```tsx
const { GraphCanvas } = useGraphViewer({
  dataModel: { space: "industrial", externalId: "EWIS", version: "1" },
  instance: { space: "instances", externalId: "connector-001" },
  options: {
    whitelistedRelationProps: ["parent", "child", "connectedTo"],
    coreReverseQueries: [
      ["industrial-dm", "Cavity", "connector", false],
      ["industrial-dm", "Cable", "wireGroup", true],
    ],
    initialConnectionLimit: 50,
    maxNodes: 500,
  },
});
```

## Sizing

The `<GraphCanvas>` fills its parent. Give the parent explicit dimensions:

```tsx
// Fixed height
<GraphCanvas className="h-[600px] w-full" />

// Fill parent
<div className="h-full w-full">
  <GraphCanvas className="h-full w-full" />
</div>

// Flex child
<div className="flex flex-col h-screen">
  <header>...</header>
  <GraphCanvas className="flex-1" />
</div>
```

## Common Patterns

### Reacting to selection

```tsx
const { GraphCanvas, selectedNode } = useGraphViewer({ ... });

useEffect(() => {
  if (selectedNode) {
    console.log("Selected:", selectedNode.data.externalId);
  }
}, [selectedNode]);
```

### Expanding on external trigger

```tsx
const { expandNode } = useGraphViewer({ ... });

// nodeId format is "space:externalId"
await expandNode("my-space:pump-001");
```

---

## Appendix: Componentization Pattern

This section describes the general pattern used to extract `useGraphViewer` from the full `GraphViewer` component. Apply the same approach when extracting any reusable hook from a larger codebase.

### 1. Classify source code into buckets

| Bucket | Keep in lite? | Examples |
|--------|--------------|---------|
| **Core** | Yes | Canvas rendering, data transforms, buffer, selection |
| **Integration** | No | Chat sidebar, diagram viewer, saved scenes, search UI |
| **Shared** | Import as-is | Types, service functions, config defaults, sub-components |

### 2. Design the hook API

- Inputs are **domain-level** (`{ space, externalId, version }`) not internal structures.
- Return a **self-contained render component** plus state and imperative controls.
- Group advanced options under an `options` object with sensible defaults.

### 3. Decompose into sub-hooks

| Sub-hook | Responsibility |
|----------|---------------|
| `useResourceLoader` | Fetch the domain resource from the API |
| `useSeedLoader` | Load + expand an initial item into the buffer |
| `useBuffer` | Manage the in-memory node/edge collection (reuse existing) |
| `useSelection` | Track selected items (reuse existing) |
| `useDataPipeline` | Transform buffer to render format (reuse existing) |

The main hook composes them, wires interaction callbacks, and builds the component via `useMemo` for stable identity.

### 4. File structure

```
feature/lite/
  index.ts                # Public exports (hook + types only)
  types.ts                # Config, return type, feature flags
  useFeature.tsx          # Main hook (composes sub-hooks, returns component)
  useResourceLoader.ts    # Fetches domain resource
  useSeedLoader.ts        # Loads initial data into buffer
  FeatureCanvas.tsx       # Lightweight render component (props-driven)
```

### 5. Checklist

- Hook input uses domain-level identifiers, not internal types
- Returned component is self-contained (consumer just renders it)
- Shared code is imported, never duplicated
- Integration features are excluded from the lite version
- Feature flags allow toggling overlays (legend, zoom, expansion)
- `useMemo` wraps the component for stable identity
- Public export surface is minimal (hook + types only)
