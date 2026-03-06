---
name: code-quality
description: "MUST be used whenever reviewing a Dune app for code quality, maintainability, or clean code issues — before a PR review, after a feature is complete, or when the user asks for a code review. Do NOT skip linting steps. Triggers: code quality, code review, clean code, refactor, maintainability, technical debt, any type, naming, dead code, duplication, DRY, single responsibility, component size, lint, linting, TypeScript strict, dependency injection, file structure."
allowed-tools: Read, Glob, Grep, Shell, Write
metadata:
  argument-hint: "[file, directory, or PR branch to review — e.g. 'src/components/AssetPanel.tsx']"
---

# Code Quality Review

Review **$ARGUMENTS** (or the whole app if no argument is given) for code quality issues. Work through every step below in order and report all findings with file paths and line numbers.

---

## Step 1 — Run the linter first

Before reading any code manually, get a baseline from the automated tools:

```bash
pnpm run lint
```

List every error and warning. Fix all errors before proceeding — lint errors are not negotiable. Warnings should be reviewed and resolved unless there is a documented exception.

Also run the TypeScript compiler in strict mode to surface any hidden type issues:

```bash
pnpm exec tsc --noEmit
```

List every type error. These must be fixed.

---

## Step 2 — Eliminate `any` types

Search for `any` usage across the codebase:

```bash
grep -rn --include="*.ts" --include="*.tsx" -E ": any|as any|<any>" src/
```

For each hit, replace with the correct type. Common substitutions:

| Instead of | Use |
|------------|-----|
| `any` for unknown external data | `unknown` + type guard or Zod parse |
| `any` for event handlers | `React.ChangeEvent<HTMLInputElement>`, `React.MouseEvent`, etc. |
| `any` for CDF responses | The SDK's own response types (import from `@cognite/sdk`) |
| `any[]` for arrays | `T[]` with the correct generic |
| `as any` casts | Proper type narrowing or explicit overloaded function signature |

The goal is zero `any` in `src/`. If a third-party library forces it, wrap the call in a typed adapter function so `any` does not leak into the app.

---

## Step 3 — Check component size and single responsibility

List all `.tsx` files with their line counts:

```bash
Get-ChildItem -Recurse -Filter "*.tsx" src | ForEach-Object { [PSCustomObject]@{ Lines = (Get-Content $_.FullName | Measure-Object -Line).Lines; Name = $_.FullName } } | Sort-Object Lines -Descending | Select-Object Lines, Name
```

Flag every component file over **150 lines**. For each, read it and check:

- Does it do more than one thing? (fetch data AND render UI AND handle form state)
- Can the fetch logic move to a custom hook (`useAssetData`)?
- Can sub-sections be extracted as named sub-components?

Apply the split only when it creates a genuinely cleaner separation — do not split for the sake of line count alone. A well-named 200-line component is better than three poorly-named 60-line ones.

---

## Step 4 — Find and remove duplicate logic (DRY)

Search for copy-pasted patterns across hooks, utilities, and components:

```bash
# Find repeated fetch patterns
grep -rn --include="*.ts" --include="*.tsx" -E "sdk\.(assets|timeseries|events|files)\.(list|retrieve)" src/

# Find repeated formatting functions
grep -rn --include="*.ts" --include="*.tsx" -E "toLocaleDateString|toLocaleString|new Date\(" src/

# Find repeated className strings longer than 40 chars
grep -rn --include="*.tsx" -E 'className="[^"]{40,}"' src/
```

For each set of duplicates:
- Extract to `src/utils/` if it is a pure function
- Extract to `src/hooks/` if it contains React state or effects
- Extract to a shared component if it is JSX

---

## Step 5 — Enforce dependency injection for external calls

Components and hooks must not import the CDF client directly. The SDK client must be obtained from context (via `useCogniteClient()` or a prop) so the component is testable in isolation.

```bash
grep -rn --include="*.ts" --include="*.tsx" -E "new CogniteClient|createCogniteClient" src/
```

Flag any direct client construction outside of the app's bootstrap / auth setup file. The pattern should always be:

```ts
// GOOD — client comes from context
export function useMyData() {
  const sdk = useCogniteClient(); // from Dune auth context
  // ...
}

// BAD — direct construction inside a hook or component
const sdk = new CogniteClient({ project: "my-project", ... });
```

Similarly, Atlas tools should receive their dependencies via `execute`'s closure over a hook-provided ref, not by importing a global singleton.

---

## Step 6 — Check naming conventions

Read a representative sample of files and verify:

| Artifact | Convention | Examples |
|----------|-----------|---------|
| Files & directories | `kebab-case` | `asset-panel.tsx`, `use-asset-data.ts` |
| React components | `PascalCase` | `AssetPanel`, `NavigationBar` |
| Variables, functions, hooks | `camelCase` | `isLoading`, `fetchAssets`, `useAssetData` |
| Constants (module-level) | `SCREAMING_SNAKE_CASE` | `MAX_ITEMS`, `AGENT_EXTERNAL_ID` |
| TypeScript types & interfaces | `PascalCase` | `AssetNode`, `ChartConfig` |
| Boolean variables | Auxiliary verb prefix | `isLoading`, `hasError`, `canEdit` |

Search for common violations:

```bash
# Components not in PascalCase files
Get-ChildItem -Recurse -Filter "*.tsx" src | Where-Object { $_.Name -cmatch "^[a-z]" }

# Hook files not prefixed with "use"
Get-ChildItem -Recurse -Filter "*.ts" src/hooks | Where-Object { $_.Name -notmatch "^use" }
```

---

## Step 7 — Remove dead code

```bash
# Find commented-out code blocks (3+ consecutive commented lines)
grep -rn --include="*.tsx" --include="*.ts" -E "^\s*//" src/ | uniq -c | awk '$1 >= 3'

# Find console.log/debug statements
grep -rn --include="*.tsx" --include="*.ts" -E "console\.(log|debug|warn|error|info)" src/

# Find TODO/FIXME/HACK comments
grep -rn --include="*.tsx" --include="*.ts" -E "(TODO|FIXME|HACK|XXX):" src/
```

Rules:
- `console.log` and `console.debug` must be removed before shipping (use proper error logging for `console.error`).
- Commented-out code blocks must be removed — version control preserves history.
- `TODO` and `FIXME` comments older than the current sprint should be resolved or converted to tracked issues.
- Unused imports are caught by the linter (Step 1); confirm they are gone.

---

## Step 8 — Verify file and export structure

Every feature area should follow a consistent structure. Check that the app's layout matches this pattern:

```
src/
├── components/         # Shared presentational components
│   └── <name>/
│       ├── <name>.tsx
│       └── index.ts    # re-exports the public API
├── hooks/              # Custom hooks (each file = one hook)
├── utils/              # Pure utility functions (no React)
├── contexts/           # React context providers
├── pages/ or views/    # Route-level components
└── types/              # Shared TypeScript types
```

Flag:
- Business logic sitting directly in page components (should be in hooks)
- Utility functions living inside component files (should be in `utils/`)
- Types defined inline in component files when they are used across multiple files (should be in `types/`)
- Missing `index.ts` barrel files for component directories (makes imports verbose)

---

## Step 9 — Report findings

Produce a structured report grouped by category:

| Category | File | Line | Issue | Recommendation |
|----------|------|------|-------|----------------|
| TypeScript | `src/hooks/useData.ts` | 18 | `response as any` cast | Import and use `NodeItem` type from `@cognite/sdk` |
| Size | `src/components/Dashboard.tsx` | — | 340 lines, mixes fetch and render logic | Extract `useDashboardData` hook (~120 lines) |
| DRY | `src/components/A.tsx`, `src/components/B.tsx` | 45, 62 | Identical date formatter | Extract to `src/utils/formatDate.ts` |
| Naming | `src/hooks/data.ts` | — | File name does not start with `use` | Rename to `useData.ts` |
| Dead code | `src/App.tsx` | 88 | `console.log("debug response", data)` | Remove |

If no issues are found in a step, state "No issues found" for that step. Do not skip steps silently.

---

## Done

Summarize the total number of findings by category and list the highest-impact items to address first. Any `any` type and lint error must be treated as blocking — list these separately.
