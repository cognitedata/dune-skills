---
name: setup-dune-auth
description: "MUST be used when migrating an existing React app to Dune, or when no Dune auth provider wraps the app. Detects classic vs Apps API flow from `app.json` `infra` field, installs the right packages, wires up the entry file, and exposes `useDune()`. No-op when the correct provider is already in place. Triggers: migrate to Dune, add Dune auth, DuneAuthProvider, AppSdkAuthProvider, useDune, Dune setup, setup auth, missing auth provider, CDF authentication, Fusion iframe auth."
allowed-tools: Read, Glob, Grep, Edit, Write, Bash
metadata:
  argument-hint: ""
---

# Set Up Dune Authentication

Wire a React app for Dune auth so `useDune()` returns an authenticated `CogniteClient`. Two flows exist; pick one based on `app.json`.

## Pick the flow

Read `app.json` if present:

| `app.json` `infra` | Flow | Auth provider | Extra package |
|---|---|---|---|
| `"appsApi"` | **Apps API** (new Fusion app host) | `AppSdkAuthProvider` | `@cognite/app-sdk` |
| missing / other | **Classic** (legacy Files API) | `DuneAuthProvider` | — |

No `app.json`? Ask the user. Default to **Apps API** — it's the default for `npx @cognite/dune create`.

Both flows expose the same `useDune()` returning `{ sdk, isLoading, error }`. Steps below are shared except where noted.

## Step 1 — Read state, skip what's done

Read `package.json`, `src/main.tsx` (or `src/index.tsx`), `vite.config.ts`, `app.json`. If the correct provider already wraps the app and deps/Vite config are in place, **do nothing** and report no-op.

Detect package manager from the lock file (`pnpm-lock.yaml` → pnpm, `yarn.lock` → yarn, otherwise npm).

## Step 2 — Install missing deps

Required for **both** flows:

| Package | Type |
|---|---|
| `@cognite/dune` | runtime |
| `@cognite/sdk` | runtime |
| `@tanstack/react-query` | runtime |
| `vite-plugin-mkcert` | dev |

**Apps API only**, also install:

| Package | Type |
|---|---|
| `@cognite/app-sdk` | runtime |

Skip anything already in `package.json`. Use the detected package manager (`pnpm add`, `npm install`, `yarn add`; `-D` / `--save-dev` for dev deps).

## Step 3 — Vite config

`vite.config.ts` must contain:

```ts
import { fusionOpenPlugin } from "@cognite/dune/vite";
import mkcert from "vite-plugin-mkcert";

export default defineConfig({
  base: "./",
  plugins: [react(), mkcert(), fusionOpenPlugin(), /* ... */],
  server: { port: 3001 },
  worker: { format: "es" },
});
```

`base: "./"` and `mkcert()` are required for Fusion iframe auth. `fusionOpenPlugin()` opens the dev URL inside Fusion. Don't remove existing plugins; only add what's missing.

## Step 4 — Wrap the app in the entry file

Edit `src/main.tsx` (or equivalent). Provider must be **inside** `QueryClientProvider`, **outside** `<App />`.

### Classic — `DuneAuthProvider`

```tsx
import { DuneAuthProvider } from "@cognite/dune";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App.tsx";

const queryClient = new QueryClient({
  defaultOptions: { queries: { staleTime: 5 * 60 * 1000, gcTime: 10 * 60 * 1000 } },
});

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <DuneAuthProvider>
        <App />
      </DuneAuthProvider>
    </QueryClientProvider>
  </React.StrictMode>
);
```

### Apps API — `AppSdkAuthProvider`

Identical to above, but swap the import and the wrapper:

```tsx
import { AppSdkAuthProvider } from "@cognite/dune";
// ...
<QueryClientProvider client={queryClient}>
  <AppSdkAuthProvider>
    <App />
  </AppSdkAuthProvider>
</QueryClientProvider>
```

If the entry currently calls `connectToHostApp` directly with manual `useState`, replace it with `AppSdkAuthProvider` — same `useDune()` API, less boilerplate.

If the correct provider already wraps the app, leave the file alone.

## Step 5 — Use `useDune()` in components

Search for manual `new CogniteClient(...)`, custom CDF auth contexts, OIDC handlers, or direct `connectToHostApp` calls in components. Replace with:

```tsx
import { useDune } from "@cognite/dune";

const { sdk, isLoading, error } = useDune();
```

`sdk` is an authenticated `CogniteClient`. Skip if there's nothing to replace.

## Step 6 — Clean up superseded code

Remove only what's now redundant:

- Custom CDF auth providers/hooks
- Manual `CogniteClient` instantiation
- OIDC/token-management code
- CDF env vars (`VITE_CDF_PROJECT`, `VITE_CDF_CLUSTER`, etc.) — Dune gets these from the host

If unsure, leave it and flag to the user.
