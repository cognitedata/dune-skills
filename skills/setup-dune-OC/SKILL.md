---
name: setup-dune-OC
description: "MUST be used when migrating an existing React app to Dune, or when DuneAuthProvider is missing from the app. This skill installs the @cognite/dune package, wraps the app in DuneAuthProvider, configures the Vite plugin, and sets up the useDune hook. Triggers: migrate to Dune, add Dune auth, DuneAuthProvider, useDune, Dune setup, setup auth, missing auth provider, CDF authentication, Fusion iframe auth."
allowed-tools: Read, Glob, Grep, Edit, Write, Bash
metadata:
  argument-hint: ""
---

# Set Up Dune Authentication

Migrate an existing React app to Dune by installing auth dependencies, wrapping the app in `DuneAuthProvider`, and configuring the Vite plugin.

## Your job

Complete these steps in order. Read each file before modifying it.

---

## Step 1 — Understand the app

Read these files to understand the current setup:

- `package.json` — detect package manager (`packageManager` field or lock file) and existing deps
- `src/main.tsx` (or `src/index.tsx`, `src/main.ts`) — find the app entry point and root render
- `vite.config.ts` (or `vite.config.js`) — check for existing Vite configuration
- `src/App.tsx` (or equivalent) — understand the app structure

Check what's already in place:

- Is `@cognite/dune` already in `package.json`?
- Is `DuneAuthProvider` already wrapping the app?
- Is `fusionOpenPlugin` already in the Vite config?
- Is `@tanstack/react-query` already installed?

Only fix what's missing — do not duplicate existing setup.

---

## Step 2 — Install dependencies

Install the required packages using the app's package manager. Only install packages that are **not** already in `package.json`.

Required dependencies:

| Package | Purpose |
|---------|---------|
| `@cognite/dune` | Auth provider, useDune hook, Vite plugin |
| `@cognite/sdk` | CogniteClient for CDF API access |
| `@tanstack/react-query` | Required peer dependency for Dune |

Required dev dependencies:

| Package | Purpose |
|---------|---------|
| `vite-plugin-mkcert` | HTTPS in local dev (required for Fusion iframe auth) |

Install commands by package manager:

- pnpm → `pnpm add @cognite/dune @cognite/sdk @tanstack/react-query && pnpm add -D vite-plugin-mkcert`
- npm  → `npm install @cognite/dune @cognite/sdk @tanstack/react-query && npm install -D vite-plugin-mkcert`
- yarn → `yarn add @cognite/dune @cognite/sdk @tanstack/react-query && yarn add -D vite-plugin-mkcert`

Skip any package that's already installed.

---

## Step 3 — Configure Vite

Edit `vite.config.ts` to add the `fusionOpenPlugin` and `mkcert` plugins if they are not already present.

### Required imports

```ts
import { fusionOpenPlugin } from "@cognite/dune/vite";
import mkcert from "vite-plugin-mkcert";
```

### Required plugin entries

Add `mkcert()` and `fusionOpenPlugin()` to the `plugins` array:

```ts
export default defineConfig({
  base: "./",
  plugins: [react(), mkcert(), fusionOpenPlugin(), /* ...other plugins */],
  server: {
    port: 3001,
  },
  worker: {
    format: "es",
  },
});
```

Key details:

- `base: "./"` is required for Fusion iframe deployment
- `mkcert()` enables HTTPS locally, which is required for Fusion iframe communication
- `fusionOpenPlugin()` opens the app inside Fusion during `pnpm dev` for auth to work
- `server.port: 3001` is the conventional Dune dev port (optional but recommended)
- `worker.format: "es"` is needed if web workers are used (optional but recommended)

Only add what's missing. Do not remove existing plugins.

---

## Step 4 — Wrap the app in DuneAuthProvider

Edit the entry file (`src/main.tsx` or equivalent) to wrap the app with `DuneAuthProvider` and `QueryClientProvider`.

### Target structure

```tsx
import { DuneAuthProvider } from "@cognite/dune";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App.tsx";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,
      gcTime: 10 * 60 * 1000,
    },
  },
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

Important rules:

- `DuneAuthProvider` must be **inside** `QueryClientProvider`
- `DuneAuthProvider` must be **outside** (wrapping) the `<App />` component
- Keep any existing providers the app already has — nest `DuneAuthProvider` around the app content but inside `QueryClientProvider`
- If `QueryClientProvider` already exists, just add `DuneAuthProvider` inside it
- If `DuneAuthProvider` already wraps the app, do nothing

---

## Step 5 — Wire up useDune in components

Search the app for any existing CogniteClient instantiation or auth setup (e.g. manual `new CogniteClient(...)`, custom auth contexts, or OIDC flows) and replace them with the `useDune` hook.

### The hook API

```tsx
import { useDune } from "@cognite/dune";

function MyComponent() {
  const { sdk, isLoading, error } = useDune();

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  // sdk is an authenticated CogniteClient — use it directly
  // e.g. sdk.timeseries.list(), sdk.assets.list(), etc.
}
```

Return values:

| Field | Type | Description |
|-------|------|-------------|
| `sdk` | `CogniteClient` | Authenticated Cognite SDK instance |
| `isLoading` | `boolean` | `true` during initial auth handshake |
| `error` | `string \| undefined` | Error message if auth fails |

If the app has no existing CDF auth code to replace, skip this step.

---

## Step 6 — Clean up old auth code

If the app previously used a custom auth setup, remove it:

- Delete custom auth context providers or hooks that are now replaced by `DuneAuthProvider` / `useDune`
- Remove manual `CogniteClient` instantiation from entry files
- Remove OIDC/token management code that `DuneAuthProvider` now handles
- Remove any environment variables for CDF credentials (e.g. `VITE_CDF_PROJECT`, `VITE_CDF_CLUSTER`) — Dune handles this via Fusion iframe messaging

Only remove code that is clearly superseded. If unsure, leave it and flag it to the user.

---

## Done

The app should now:

1. Have `@cognite/dune`, `@cognite/sdk`, and `@tanstack/react-query` installed
2. Have `DuneAuthProvider` wrapping the app in the entry file
3. Have `fusionOpenPlugin()` and `mkcert()` in the Vite config
4. Use `useDune()` to access the authenticated `CogniteClient`

Run `pnpm dev` (or the app's dev command) — it should open in Fusion and authenticate automatically.
