# Vite Configuration for @cognite/reveal

## main.tsx — Process polyfill (must be first import)

```tsx
import process from "process";
(window as unknown as Record<string, unknown>).process = process;

// ... all other imports below
```

## vite.config.ts — Full working config

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import basicSsl from "@vitejs/plugin-basic-ssl";
import path from "path";

export default defineConfig({
  base: "./",
  plugins: [react(), basicSsl()],
  define: {
    "process.env": {},
    "process.platform": JSON.stringify(""),
    "process.version": JSON.stringify(""),
    global: "globalThis",
  },
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
      util: "util/",
      assert: "assert/",
      process: "process/browser",
      three: path.resolve(__dirname, "node_modules/three/build/three.module.js"),
    },
    dedupe: ["three"],
    conditions: ["import", "module", "browser", "default"],
  },
  optimizeDeps: {
    esbuildOptions: {
      define: {
        global: "globalThis",
      },
    },
    include: ["process", "util", "assert", "three", "@cognite/reveal"],
  },
  server: {
    port: 3000,
    https: true,
  },
  worker: {
    format: "es",
  },
});
```

## Why each setting is needed

| Setting | Reason |
|---------|--------|
| `process` polyfill in main.tsx | `@cognite/reveal` deps use `process.env` at runtime |
| `define.process.env` | Vite build-time replacement for `process.env` references |
| `define.global` | Some deps use `global` instead of `globalThis` |
| `resolve.alias.util/assert/process` | Browser-compatible replacements for Node built-ins |
| `resolve.alias.three` | Single Three.js instance — prevents "Multiple instances" warning |
| `resolve.dedupe` | Ensures symlinked packages share the same `three` |
| `optimizeDeps.include` | Pre-bundles these deps so esbuild can handle CJS→ESM conversion |
| `resolve.conditions` | Ensures browser module variants are preferred |
