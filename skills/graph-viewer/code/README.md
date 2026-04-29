# GraphViewer

An interactive graph visualization component for exploring **Cognite Data Fusion (CDF)** data model instances and their relationships. Built on top of [reagraph](https://github.com/reaviz/reagraph), it provides a single hook — `useGraphViewer` — that returns a ready-to-render canvas and a full set of programmatic controls.

## Features

- **Data model-aware** — automatically loads CDF data model metadata to resolve node types, icons, and colors.
- **Progressive exploration** — starts from a seed instance and lets users expand the graph by double-clicking nodes to fetch connected instances (edges, direct relations, and configurable reverse relations).
- **LRU node buffer** — keeps the graph performant by evicting least-recently-used nodes when `maxNodes` is exceeded.
- **Multiple layouts** — Force-directed (2D/3D), tree (top-down / left-right), radial, and circular.
- **Interactive legend** — color-coded node type legend with click-to-filter.
- **Zoom controls** — built-in zoom in/out and fit-to-view buttons.
- **Theming** — fully customizable node, edge, ring, arrow, and canvas colors via `GraphThemeConfig` and `GraphVisualConfig`.
- **Type-aware icons** — maps CDF view types (ISA-95 assets, equipment, files, time series, etc.) to SVG icons rendered inside node circles.

## Quick Start

```tsx
function MyGraph() {
  const { GraphCanvas, isLoading, error } = useGraphViewer({
    dataModel: { space: "my-space", externalId: "my-dm", version: "1" },
    instance: { space: "my-inst-space", externalId: "pump-001" },
  });

  if (isLoading) return <p>Loading…</p>;
  if (error) return <p>Error: {error}</p>;

  return <GraphCanvas className="h-full w-full" />;
}
```

## API

### `useGraphViewer(config): UseGraphViewerReturn`


| Config field | Type                             | Description                               |
| ------------ | -------------------------------- | ----------------------------------------- |
| `dataModel`  | `{ space, externalId, version }` | **Required.** The CDF data model to load. |
| `instance`   | `{ space, externalId }`          | Optional seed instance to load on mount.  |
| `options`    | `UseGraphViewerOptions`          | Optional overrides (see below).           |


#### `UseGraphViewerOptions`


| Option                     | Type                              | Default             | Description                                                |
| -------------------------- | --------------------------------- | ------------------- | ---------------------------------------------------------- |
| `maxNodes`                 | `number`                          | `1000`              | Maximum nodes held in the LRU buffer.                      |
| `layout`                   | `LayoutType`                      | `"forceDirected2d"` | Initial graph layout algorithm.                            |
| `initialConnectionLimit`   | `number`                          | `100`               | Max edges fetched per expansion.                           |
| `whitelistedRelationProps` | `string[]`                        | all                 | Property names to follow when extracting direct relations. |
| `coreReverseQueries`       | `[space, viewId, prop, isList][]` | `[]`                | Reverse-relation queries to run on node expansion.         |
| `viewPriorityConfig`       | `ViewPriorityConfig`              | built-in            | Controls which CDF views determine node types.             |
| `visualConfig`             | `Partial<GraphVisualConfig>`      | defaults            | Node colors, palette, icon size, path highlight.           |
| `themeConfig`              | `Partial<GraphThemeConfig>`       | defaults            | Full reagraph theme overrides.                             |
| `features`                 | `Partial<LiteFeatureFlags>`       | all enabled         | Toggle legend, zoom controls, and node expansion.          |


#### `UseGraphViewerReturn`


| Property        | Type                                   | Description                                                    |
| --------------- | -------------------------------------- | -------------------------------------------------------------- |
| `GraphCanvas`   | `React.FC<{ className? }>`             | Self-contained canvas component to render.                     |
| `isLoading`     | `boolean`                              | `true` while data model, seed node, or expansion is in flight. |
| `error`         | `string | null`                        | Error message, if any.                                         |
| `graphData`     | `GraphData`                            | Current nodes, connections, and node type metadata.            |
| `stats`         | `GraphStats | null`                    | Aggregate counts by node/connection type.                      |
| `layout`        | `LayoutType`                           | Current layout.                                                |
| `setLayout`     | `(layout) => void`                     | Change the layout algorithm.                                   |
| `selections`    | `string[]`                             | Currently selected node/edge IDs.                              |
| `setSelections` | `(ids) => void`                        | Programmatically select nodes/edges.                           |
| `selectedNode`  | `GraphNode | null`                     | The selected node object.                                      |
| `selectedEdge`  | `GraphEdge | null`                     | The selected edge object.                                      |
| `expandNode`    | `(nodeId) => Promise<void>`            | Fetch and add connected instances for a node.                  |
| `loadInstance`  | `(space, externalId) => Promise<void>` | Load a new seed instance (replaces the graph).                 |
| `fitView`       | `() => void`                           | Fit all nodes into the viewport.                               |
| `zoomIn`        | `() => void`                           | Zoom in.                                                       |
| `zoomOut`       | `() => void`                           | Zoom out.                                                      |
| `clear`         | `() => void`                           | Remove all nodes and edges from the buffer.                    |
| `graphRef`      | `RefObject<GraphCanvasRef>`            | Direct ref to the underlying reagraph canvas.                  |


## Feature Flags

```tsx
const { GraphCanvas } = useGraphViewer({
  dataModel: { space: "s", externalId: "dm", version: "1" },
  options: {
    features: {
      enableLegend: true,        // show/hide legend (default: true)
      enableZoomControls: true,  // show/hide zoom buttons (default: true)
      enableNodeExpansion: true,  // allow double-click expand (default: true)
    },
  },
});
```

## Layout Options


| ID                | Label             |
| ----------------- | ----------------- |
| `forceDirected2d` | Force 2D          |
| `forceDirected3d` | Force 3D          |
| `treeTd2d`        | Tree (Top-Down)   |
| `treeLr2d`        | Tree (Left-Right) |
| `radialOut2d`     | Radial            |
| `circular2d`      | Circular          |


## Architecture

```
graph-viewer/
├── useGraphViewer.tsx        # Main hook — composes all sub-hooks, returns GraphCanvas + controls
├── GraphViewerCanvas.tsx     # Renders reagraph canvas, zoom controls, and legend
├── GraphViewerLegend.tsx     # Color-coded node type legend with click-to-filter
├── ZoomControls.tsx          # Zoom in / out / fit-view button group
├── graph-service.ts          # CDF API calls — fetchNodeDetails, fetchConnectedNodes
├── graph-config.ts           # Theme defaults, icon generation, node/edge transformers
├── useDataModelLoader.ts     # Loads data model views from CDF
├── useSeedNode.ts            # Loads the initial instance and its connections
├── useNodeBuffer.ts          # LRU buffer that caps total nodes at maxNodes
├── useGraphDataPipeline.ts   # Transforms raw CDF instances into GraphData + reagraph format
├── useGraphSelection.ts      # Tracks selected node/edge state
├── useCanvasResize.ts        # Observes container size changes and triggers reagraph resize
├── types.ts                  # All shared TypeScript types, constants, and helpers
└── index.ts                  # Public exports
```

## Dependencies


| Package         | Purpose                                        |
| --------------- | ---------------------------------------------- |
| `reagraph`      | WebGL graph rendering engine                   |
| `@cognite/sdk`  | CDF API client (instances, data models)        |
| `@cognite/dune` | Provides the authenticated SDK via `useDune()` |


