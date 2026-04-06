# Vite Configuration for @cognite/reveal in a Dune monorepo app

## src/dune-fe-auth-shim.ts — create this file first

```ts
// useDune and useCDF have identical signatures: { sdk, isLoading, error }
export { useDune as useCDF } from '@cognite/dune';
```

This shim is aliased in `vite.config.ts` so that when `@cognite/dune-industrial-components/reveal`
calls `useCDF()` from `dune-fe-auth`, it gets the shim instead of the real package.

**Why not install the real dune-fe-auth?**
`dune-fe-auth` was compiled against React 18 and internally calls
`React.__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED.ReactCurrentDispatcher`.
That property was removed in React 19. Dune monorepos use React 19 — the real package
crashes regardless of bundling strategy.

---

## src/main.tsx — process polyfill must be the very first two lines

```tsx
import process from 'process';
(window as unknown as Record<string, unknown>).process = process;

// All other imports below — order matters, polyfill must run first
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

Keep `DuneAuthProvider` from `@cognite/dune`. Do **not** use `CDFAuthenticationProvider`
from `dune-fe-auth` — the shim makes this unnecessary and avoids the React 18/19 conflict.

---

## vite.config.ts — standalone config (not mergeConfig)

New Dune apps generated with `pnpm new:app` have a standalone `vite.config.ts`.
Replace the file entirely with the following:

```typescript
import path from 'node:path';

import { fusionOpenPlugin } from '@cognite/dune/vite';
import tailwindcss from '@tailwindcss/vite';
import react from '@vitejs/plugin-react';
import { defineConfig } from 'vite';
import mkcert from 'vite-plugin-mkcert';

export default defineConfig({
  base: './',
  plugins: [react(), mkcert(), fusionOpenPlugin(), tailwindcss()],
  define: {
    // Some CJS deps use `global` instead of `globalThis`
    'process.env': {},
    'process.platform': JSON.stringify(''),
    'process.version': JSON.stringify(''),
    global: 'globalThis',
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),

      // --- dune-fe-auth shim ---
      // @cognite/dune-industrial-components imports useCDF from dune-fe-auth.
      // We point it to our shim (which re-exports useDune as useCDF) so the real
      // dune-fe-auth (compiled for React 18) is never loaded in this React 19 monorepo.
      'dune-fe-auth': path.resolve(__dirname, 'src/dune-fe-auth-shim.ts'),

      // --- Node built-in polyfills ---
      // Use explicit package aliases, not vite-plugin-node-polyfills.
      // The plugin introduces transitive dep conflicts ("Could not resolve 'inherits'").
      // The `process`, `util`, and `assert` packages must be in app dependencies.
      util: 'util/',
      assert: 'assert/',
      process: 'process/browser',

      // --- Single Three.js instance ---
      // @cognite/reveal bundles its own Three.js copy. Without this alias, the app
      // and Reveal load two separate copies → "Multiple instances of Three.js" warning
      // and broken rendering. Requires `three` as a direct app dependency.
      three: path.resolve(__dirname, 'node_modules/three/build/three.module.js'),
    },

    // pnpm uses a virtual store with symlinks. Packages that symlink to different
    // paths can resolve to separate module instances even for the same package.
    // `dedupe` forces a single physical copy for all pre-bundles.
    // Missing `react`/`react-dom` here causes ReactCurrentDispatcher errors.
    // Missing `three` causes "Multiple instances" warnings.
    dedupe: ['react', 'react-dom', 'react/jsx-runtime', '@tanstack/react-query', 'three'],

    conditions: ['import', 'module', 'browser', 'default'],
  },
  optimizeDeps: {
    // NEVER add @cognite/dune-industrial-components or dune-fe-auth to `exclude`.
    // Excluding forces raw ESM serving. In pnpm's virtual store, raw ESM files resolve
    // React via their own physical path — a different module instance from the
    // pre-bundled singleton — causing ReactCurrentDispatcher errors even with dedupe.
    // Let Vite auto-discover and pre-bundle them together with everything else.
    esbuildOptions: {
      define: { global: 'globalThis' },
    },
    include: [
      // Vite can't auto-discover bare polyfill imports (no source file imports them
      // directly). List them explicitly so esbuild pre-bundles them.
      'process',
      'util',
      'assert',
      // Heavy/complex deps — explicit listing speeds up cold starts
      'three',
      '@cognite/reveal',
      // React ecosystem — pre-bundling creates the CJS→ESM singleton all deps share.
      // If any of these are missing, a dep that imports React raw can get a second copy.
      'react',
      'react-dom',
      '@tanstack/react-query',
    ],
  },
  server: {
    port: 3002,
  },
  worker: {
    // @cognite/reveal spawns ES module web workers. Without 'es' format they fail
    // silently — black screen with no console error.
    format: 'es',
  },
});
```

---

## Why each setting is needed

| Setting | Reason |
|---------|--------|
| `dune-fe-auth` alias → shim | Prevents React 18 internals code from ever loading in a React 19 monorepo |
| `util/`, `assert/`, `process/browser` aliases | Browser-compatible replacements for Node built-ins. Packages must be in `dependencies`. Do NOT use `vite-plugin-node-polyfills` — causes "Could not resolve 'inherits'" |
| `process` polyfill in main.tsx first | `@cognite/reveal` deps call `process.env` at **runtime** (not build-time). The `define` replacements handle build-time; the window assignment handles runtime |
| `define.global = 'globalThis'` | Some CJS deps use `global` instead of `globalThis` |
| `resolve.alias.three` | Single Three.js instance — without this, Reveal's bundled copy and the app's copy conflict |
| `resolve.dedupe` with react + react-dom + react/jsx-runtime | pnpm symlinks can create separate module instances. `dedupe` forces one copy. Critical — missing these causes `ReactCurrentDispatcher` errors |
| `resolve.dedupe` with three | Ensures symlinked packages (e.g. `@cognite/dune-industrial-components`) share the same Three.js |
| `optimizeDeps.include` for process/util/assert | No source file imports them, so Vite cannot auto-discover them for pre-bundling |
| `optimizeDeps.include` for react + react-dom + @tanstack/react-query | Converts CJS → ESM and creates a single shared instance. All pre-bundled deps that import React get the same copy |
| `worker.format: 'es'` | Reveal spawns ES module workers; Vite defaults to IIFE/UMD which breaks them |
| `conditions: ['import', 'module', 'browser', 'default']` | Ensures browser ESM variants are preferred over CJS/Node variants |

## Common mistakes that break the setup

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Running `pnpm install` in Cursor sandbox without `required_permissions: ["all"]` | `git init ... Operation not permitted` — pnpm can't clone the GitHub package | Add `required_permissions: ["all"]` to the Shell tool call |
| Installing `three` without checking `@cognite/reveal`'s peer requirement | `unmet peer three@0.180.0: found 0.177.x` warning; potential rendering bugs | After install, compare versions; `pnpm add three@^<peer-version>` if mismatched |
| Not adding `ajv` as a direct dependency | `unmet peer ajv@>=8: found 6.x` | `pnpm add ajv` (installs `^8`) in the app |
| Install real `dune-fe-auth` package | `ReactCurrentDispatcher` undefined — can't be fixed with any bundling strategy | Do not install it; use the shim in Step 2 |
| `@cognite/dune-industrial-components` in `optimizeDeps.exclude` | `ReactCurrentDispatcher` undefined or `No QueryClient set` | Remove from `exclude` — Vite auto-discovers it |
| Shim in `optimizeDeps.exclude` | `ReactCurrentDispatcher` undefined | Remove from `exclude` |
| `vite-plugin-node-polyfills` instead of manual aliases | `Could not resolve "inherits"` on transitive deps | Remove the plugin; add `util`, `assert`, `process` to dependencies and use aliases |
| `RevealKeepAlive` inside conditional component | `ObjectUnsubscribedError: object unsubscribed` at model load | Move `CacheProvider` + `RevealKeepAlive` to always-mounted app/page level |
| Inline arrow as `onSelect`/`onLoad` prop | `Maximum update depth exceeded` | `useCallback` at call site; call `onSelect` from `useEffect` inside model browser, never from render |
| Model browser calls `onSelect` during render (`if (revision) onSelect(...)`) | `Maximum update depth exceeded` | Move to `useEffect([revision, onSelect])` |
| Missing `worker.format: 'es'` | Black screen, no error | Add `worker: { format: 'es' }` |
| `react`/`react-dom` missing from `resolve.dedupe` | `ReactCurrentDispatcher` in pnpm monorepo | Add both to `dedupe` |
| Container has no height | Canvas collapses to 0px, nothing renders | Add `height: '70vh'` (or flex/grid height) to the parent element |
