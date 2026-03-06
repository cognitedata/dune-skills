---
name: integrate-file-viewer
description: "MUST be used whenever adding file preview/browsing to a Dune app using CogniteFileViewer. Do NOT manually wire up react-pdf, pdfjs workers, or CogniteFileViewer — this skill handles installation, Vite config, worker setup, file listing, pagination, zoom, rotation, and annotation navigation. Triggers: file viewer, file preview, CogniteFileViewer, PDF viewer, browse files, view CDF files, file explorer, document viewer."
allowed-tools: Read, Glob, Grep, Edit, Write, Bash
---

# Integrate CogniteFileViewer

Add a file explorer with PDF/image/video/text preview to this Dune app using `CogniteFileViewer` from `@cognite/dune-industrial-components/file-viewer`.

## Your job

Complete these steps in order. Read each file before modifying it.

---

## Step 1 — Understand the app

Read these files before touching anything:

- `package.json` — detect package manager (`packageManager` field or lock file) and existing deps
- `vite.config.ts` — understand current Vite setup
- `src/main.tsx` — understand entry point
- `src/App.tsx` (or equivalent entry component) — understand current structure

---

## Step 2 — Install dependencies

Install `react-pdf` and `pdfjs-dist`. They must be at matching versions — `react-pdf` declares the exact `pdfjs-dist` version it needs.

Using the app's package manager, install `react-pdf` first, then pin `pdfjs-dist` to the version react-pdf expects:

```bash
# pnpm
pnpm add react-pdf
PDFJS_VERSION=$(cat node_modules/react-pdf/package.json | grep '"pdfjs-dist"' | head -1 | grep -oE '[0-9]+\.[0-9]+\.[0-9]+')
pnpm add pdfjs-dist@$PDFJS_VERSION

# npm
npm install react-pdf
PDFJS_VERSION=$(cat node_modules/react-pdf/package.json | grep '"pdfjs-dist"' | head -1 | grep -oE '[0-9]+\.[0-9]+\.[0-9]+')
npm install pdfjs-dist@$PDFJS_VERSION

# yarn
yarn add react-pdf
PDFJS_VERSION=$(cat node_modules/react-pdf/package.json | grep '"pdfjs-dist"' | head -1 | grep -oE '[0-9]+\.[0-9]+\.[0-9]+')
yarn add pdfjs-dist@$PDFJS_VERSION
```

> **Why pin pdfjs-dist?** The API (loaded by react-pdf) and the Worker must be at identical versions. If they differ, you'll get a runtime error: `"The API version X does not match the Worker version Y"`.

---

## Step 3 — Configure Vite

Add to `vite.config.ts`:

1. An alias for the file-viewer (if consuming from the local monorepo source) **or** skip the alias if using the published package.
2. `optimizeDeps.exclude: ['pdfjs-dist']` — prevents Vite from pre-bundling pdfjs-dist, which would break the worker file path.

```ts
// vite.config.ts
export default defineConfig({
  // ... existing config ...
  optimizeDeps: {
    exclude: ['pdfjs-dist'],
  },
});
```

---

## Step 4 — Set up the PDF.js worker

In `src/main.tsx`, import the worker using Vite's `?url` suffix (so Vite resolves the path through node_modules at build time) and set it **after** all other imports so it overrides any worker URL set by the library itself at module-init time.

```ts
// src/main.tsx
import { pdfjs } from 'react-pdf';
import workerUrl from 'pdfjs-dist/build/pdf.worker.min.mjs?url';
pdfjs.GlobalWorkerOptions.workerSrc = workerUrl;

// ... rest of imports and ReactDOM.createRoot(...)
```

> **Why `?url` and why at the top?** `new URL('pdfjs-dist/...', import.meta.url)` is a plain URL concatenation — it does not go through node_modules resolution and produces a broken path. `?url` is Vite's mechanism to resolve a package file and return the correct served URL. Placing it before other imports ensures it runs last (ES module body order), overriding any worker URL set by library code during module initialisation.

---

## Step 5 — Build the file explorer UI

Replace (or update) the main `App.tsx` with a two-panel file explorer. The component must:

1. **List files** using `sdk.files.list().autoPagingToArray({ limit: -1 })` via TanStack Query (already set up in Dune apps). `limit: -1` fetches all pages.
2. **Import** `CogniteFileViewer` and `DocumentAnnotation` type from `@cognite/dune-industrial-components/file-viewer`.
3. **Prefer `instanceId` source** — if `file.instanceId` exists on the `FileInfo` object, use `{ type: 'instanceId', space, externalId }`. Fall back to `{ type: 'internalId', id }`. This is critical for annotations: the viewer only enables the annotation overlay when it can resolve an `instanceId`.
4. **Reset page, zoom, and rotation** to their defaults whenever the user selects a different file.
5. **Pagination** — wire `page`, `onPageChange`, and `onDocumentLoad` for multi-page PDFs. Only show pagination controls once `numPages` is known (`numPages > 0`).
6. **Zoom** — wire `zoom` and `onZoomChange`. Add `−`, `%` (click to reset), `+` buttons.
7. **Rotation** — wire `rotation`. Add a `↻` button that cycles `0 → 90 → 180 → 270 → 0`.
8. **Annotation navigation** — wire `onAnnotationClick`. When a diagram annotation is clicked, look up the linked file in the already-loaded list by matching `annotation.linkedResource` (`space` + `externalId`) against `file.instanceId`, then call `navigateToFile(linked)`.

### Key component API

```ts
import { CogniteFileViewer } from '@cognite/dune-industrial-components/file-viewer';
import type { DocumentAnnotation } from '@cognite/dune-industrial-components/file-viewer';

<CogniteFileViewer
  // Source (prefer instanceId when available — required for annotations)
  source={
    file.instanceId
      ? { type: 'instanceId', space: file.instanceId.space, externalId: file.instanceId.externalId }
      : { type: 'internalId', id: file.id }
  }
  client={sdk}

  // Pagination (PDFs only)
  page={page}                                         // 1-indexed, controlled
  onPageChange={setPage}
  onDocumentLoad={({ numPages }) => setNumPages(numPages)}

  // Zoom & pan (all file types)
  zoom={zoom}                                         // 1 = 100%
  onZoomChange={setZoom}
  // Ctrl/Cmd+wheel and middle-click drag are built in

  // Rotation (PDFs and images)
  rotation={rotation}                                 // 0 | 90 | 180 | 270

  // Annotation click → navigate to linked file
  onAnnotationClick={handleAnnotationClick}

  style={{ width: '100%', height: '100%' }}
/>
```

### Annotation click handler pattern

```ts
const handleAnnotationClick = useCallback((annotation: DocumentAnnotation) => {
  if (!annotation.linkedResource || !files) return;
  const { space, externalId } = annotation.linkedResource;
  const linked = files.find(
    (f) => f.instanceId?.space === space && f.instanceId?.externalId === externalId,
  );
  if (linked) navigateToFile(linked);
}, [files, navigateToFile]);
```

### Minimal skeleton

```tsx
import { useDune } from '@cognite/dune';
import { useQuery } from '@tanstack/react-query';
import { useCallback, useState } from 'react';
import { CogniteFileViewer } from '@cognite/dune-industrial-components/file-viewer';
import type { DocumentAnnotation } from '@cognite/dune-industrial-components/file-viewer';
import type { FileInfo } from '@cognite/sdk';

function App() {
  const { sdk, isLoading: sdkLoading } = useDune();
  const [selectedFile, setSelectedFile] = useState<FileInfo | null>(null);
  const [page, setPage] = useState(1);
  const [numPages, setNumPages] = useState(0);
  const [zoom, setZoom] = useState(1);
  const [rotation, setRotation] = useState<0 | 90 | 180 | 270>(0);

  const { data: files, isLoading: filesLoading } = useQuery({
    queryKey: ['cdf-files'],
    queryFn: () => sdk.files.list().autoPagingToArray({ limit: -1 }),
    enabled: !sdkLoading,
  });

  const navigateToFile = useCallback((file: FileInfo) => {
    setSelectedFile(file);
    setPage(1);
    setNumPages(0);
    setZoom(1);
    setRotation(0);
  }, []);

  const handleAnnotationClick = useCallback((annotation: DocumentAnnotation) => {
    if (!annotation.linkedResource || !files) return;
    const { space, externalId } = annotation.linkedResource;
    const linked = files.find(
      (f) => f.instanceId?.space === space && f.instanceId?.externalId === externalId,
    );
    if (linked) navigateToFile(linked);
  }, [files, navigateToFile]);

  return (
    <div style={{ display: 'flex', height: '100vh' }}>
      {/* Sidebar — file list */}
      <aside style={{ width: 280, overflowY: 'auto', borderRight: '1px solid #e5e7eb' }}>
        {filesLoading && <div>Loading files…</div>}
        {files?.map((file) => (
          <button key={file.id} type="button" onClick={() => navigateToFile(file)}>
            {file.name}
          </button>
        ))}
      </aside>

      {/* Main — toolbar + viewer */}
      <main style={{ flex: 1, display: 'flex', flexDirection: 'column', overflow: 'hidden' }}>
        {selectedFile && (
          <>
            {/* Toolbar */}
            <div style={{ display: 'flex', gap: 8, padding: '8px 16px', borderBottom: '1px solid #e5e7eb' }}>
              {/* Pagination */}
              {numPages > 0 && (
                <>
                  <button type="button" disabled={page <= 1} onClick={() => setPage((p) => p - 1)}>‹</button>
                  <span>{page} / {numPages}</span>
                  <button type="button" disabled={page >= numPages} onClick={() => setPage((p) => p + 1)}>›</button>
                </>
              )}
              {/* Rotation */}
              <button type="button" onClick={() => setRotation((r) => ((r + 90) % 360) as 0 | 90 | 180 | 270)}>↻</button>
              {/* Zoom */}
              <button type="button" onClick={() => setZoom((z) => Math.max(0.25, z * 0.8))}>−</button>
              <button type="button" onClick={() => setZoom(1)}>{Math.round(zoom * 100)}%</button>
              <button type="button" onClick={() => setZoom((z) => Math.min(5, z * 1.25))}>+</button>
            </div>

            {/* Viewer */}
            <div style={{ flex: 1, overflow: 'auto', padding: 24 }}>
              <CogniteFileViewer
                source={
                  selectedFile.instanceId
                    ? { type: 'instanceId', space: selectedFile.instanceId.space, externalId: selectedFile.instanceId.externalId }
                    : { type: 'internalId', id: selectedFile.id }
                }
                client={sdk}
                page={page}
                onPageChange={setPage}
                onDocumentLoad={({ numPages: n }) => setNumPages(n)}
                zoom={zoom}
                onZoomChange={setZoom}
                rotation={rotation}
                onAnnotationClick={handleAnnotationClick}
                style={{ width: '100%', height: '100%' }}
              />
            </div>
          </>
        )}
      </main>
    </div>
  );
}

export default App;
```

---

## Common pitfalls

| Problem | Cause | Fix |
|---|---|---|
| `"API version X does not match Worker version Y"` | `pdfjs-dist` installed at a different version than what `react-pdf` expects | Run `cat node_modules/react-pdf/package.json \| grep pdfjs-dist` and reinstall `pdfjs-dist` at that exact version |
| Worker URL resolves to `/src/pdfjs-dist/...` (404) | Using `new URL('pdfjs-dist/...', import.meta.url)` instead of `?url` | Use `import workerUrl from 'pdfjs-dist/build/pdf.worker.min.mjs?url'` |
| Annotations never show | `instanceId` is `undefined` — viewer disables annotation query when it can't resolve an instance ID | Use `instanceId` source type instead of `internalId` when `file.instanceId` exists |
| Annotations show but are empty | File has no `CogniteDiagramAnnotation` edges in CDF | Expected — only P&ID / diagram files synced to the data model have annotations |
| Left-click drag pans (blocks annotation clicks) | Upstream component fires `onMouseDown` on any button | Should be middle-click only (`e.button !== 1` guard in `handleMouseDown`) — check the component version |

---

## Done

The app should now show a file list on the left and a live preview on the right with pagination, zoom, rotation, and annotation-click navigation.
