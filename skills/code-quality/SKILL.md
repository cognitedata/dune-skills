---
name: code-quality
description: Review a Dune app for code quality, maintainability, and clean code issues. Use this skill whenever the user asks for a code review, refactoring guidance, or cleanup pass, or when they mention technical debt, naming issues, dead code, large components, duplication, TypeScript strictness, or file structure — even if they just say "clean this up" or "review my code."
---

# Code Quality Review

Review **$ARGUMENTS** (or the whole app if no argument is given) for code quality
issues. Work through every section below and report findings with file paths and
line numbers.

## Why This Matters

Code quality issues compound over time. A few `any` casts become dozens. A 300-line
component becomes a 600-line component. Duplicated logic drifts out of sync. These
issues don't cause bugs today, but they make every future change slower, riskier,
and harder to review. A periodic quality pass keeps the codebase in a state where
new features can be added confidently.

---

## Step 1 — Run Automated Checks First

Before reading any code manually, let the tooling surface the easy wins:

```bash
pnpm run lint 2>&1
```

List every error and warning. Lint errors should be fixed before proceeding — they
represent violations of rules the team has already agreed on.

Also run the TypeScript compiler to surface type issues:

```bash
pnpm exec tsc --noEmit 2>&1
```

List every type error. These indicate places where the code's actual behaviour
doesn't match its declared types — a common source of subtle bugs.

---

## Step 2 — Eliminate `any` Types

`any` disables TypeScript's type checking for everything it touches. A single
`any` can silently propagate through an entire call chain, making the type system
unable to catch bugs downstream.

```bash
rg -n ": any|as any|<any>" --type ts src/
```

For each hit, replace with the correct type:

| Instead of | Use |
|------------|-----|
| `any` for unknown external data | `unknown` + type guard or Zod parse |
| `any` for event handlers | `React.ChangeEvent<HTMLInputElement>`, `React.MouseEvent`, etc. |
| `any` for CDF responses | The SDK's own response types (import from `@cognite/sdk`) |
| `any[]` for arrays | `T[]` with the correct generic |
| `as any` casts | Proper type narrowing or an explicit overloaded function signature |

The goal is zero `any` in `src/`. If a third-party library forces it, wrap the
call in a typed adapter function so `any` does not leak into the rest of the app.

---

## Step 3 — Check Component Size and Responsibility

Large components that mix data fetching, state management, and rendering are hard
to test, review, and reuse. They also tend to re-render more than necessary because
every concern shares the same render cycle.

```bash
rg --files --type ts src/ | xargs wc -l 2>/dev/null | sort -rn | head -20
```

Flag component files over ~150 lines and read each one. Check:

- Does it mix concerns? (fetch data AND render UI AND handle form state)
- Can the fetch logic move to a custom hook (e.g., `useAssetData`)?
- Can subsections be extracted as named sub-components?

Split only when it creates a genuinely cleaner separation — a well-named 200-line
component is better than three poorly-named 60-line ones.

---

## Step 4 — Find and Remove Duplicate Logic

Duplicated code drifts out of sync over time — one copy gets a bug fix, the others
don't. Search for common duplication patterns:

```bash
rg -n "sdk\.(assets|timeseries|events|files)\.(list|retrieve)" --type ts src/
rg -n "toLocaleDateString|toLocaleString|new Date\(" --type ts src/
rg -n "className=\"[^\"]{40,}\"" --type ts src/
```

For each set of duplicates:

| Duplicate type | Extract to |
|---------------|------------|
| Pure function (formatting, calculation) | `src/utils/` |
| Logic with React state or effects | `src/hooks/` |
| Repeated JSX structure | Shared component in `src/components/` |

---

## Step 5 — Verify SDK Client Usage

Components and hooks that construct their own CDF client are untestable in
isolation — you can't mock the SDK without intercepting the constructor. The
client should come from the Dune auth context so tests can provide a mock.

```bash
rg -n "new CogniteClient|createCogniteClient" --type ts src/
```

Flag any direct client construction outside the app's bootstrap/auth setup file:

```ts
// Problem: untestable, can't mock the client
const sdk = new CogniteClient({ project: "my-project" });

// Fix: client comes from context
export function useMyData() {
  const sdk = useCogniteClient();
  // ...
}
```

Similarly, Atlas tools should receive dependencies via the `execute` closure over
a hook-provided ref, not by importing a global singleton.

---

## Step 6 — Check Naming Conventions

Inconsistent naming forces every reader to pause and decode intent. Read a
representative sample of files and verify:

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
rg --files --type ts src/hooks/ | rg -v "^.*use"
rg --files --glob "*.tsx" src/ | rg "^.*[/\\\\][a-z].*\.tsx$"
```

---

## Step 7 — Remove Dead Code

Dead code creates noise — it misleads readers into thinking it's still relevant
and makes search results harder to scan.

```bash
rg -n "console\.(log|debug)" --type ts src/
rg -n "(TODO|FIXME|HACK|XXX):" --type ts src/
```

Also search for commented-out code blocks (3+ consecutive comment lines are
usually dead code, not documentation):

```bash
rg -n "^\s*//" --type ts src/
```

Scan the results for clusters of consecutive commented lines.

Guidelines:

- **`console.log` / `console.debug`** — remove before shipping. Use structured
  error logging for `console.error` if needed for production diagnostics.
- **Commented-out code** — delete it. Version control preserves history; commented
  code just creates confusion about whether it's still needed.
- **`TODO` / `FIXME`** — resolve or convert to tracked issues. Stale TODOs
  become invisible over time.
- **Unused imports** — should already be caught by the linter (Step 1). Confirm
  they're gone.

---

## Step 8 — Verify File and Export Structure

A consistent structure makes the codebase navigable — developers know where to
find things without searching. Check that the app follows a pattern like:

```
src/
├── components/         # Shared presentational components
│   └── <name>/
│       ├── <name>.tsx
│       └── index.ts    # Re-exports the public API
├── hooks/              # Custom hooks (one hook per file)
├── utils/              # Pure utility functions (no React)
├── contexts/           # React context providers
├── pages/ or views/    # Route-level components
└── types/              # Shared TypeScript types
```

Flag:

- Business logic sitting in page components (should be in hooks or utils)
- Utility functions defined inside component files (should be in `utils/`)
- Types defined inline when they're used across multiple files (should be in
  `types/`)
- Missing `index.ts` barrel files for component directories (forces verbose
  import paths)

---

## Report Findings

Use this exact template for the report:

### Code Quality Review — [App/Component Name]

**Summary**: [1-2 sentence overview of code health and highest-impact improvements]

| Category | File | Line | Issue | Recommended Fix |
|----------|------|------|-------|-----------------|
| TypeScript | `path/to/file` | N | Description | How to fix |
| Size | `path/to/file` | — | Description | How to fix |
| DRY | `path/to/file` | N | Description | How to fix |
| Naming | `path/to/file` | — | Description | How to fix |
| Dead code | `path/to/file` | N | Description | How to fix |
| Structure | `path/to/file` | — | Description | How to fix |

### Steps with no issues found
- [List any steps where no issues were found]

If no issues are found in a step, say so explicitly — skipping steps silently
makes it unclear whether the step was checked.

After presenting the report, offer to fix the issues directly — starting with
`any` types and lint errors, which are the most mechanical to resolve.

---

## Gotchas

1. **Don't over-abstract.** Extracting a utility function used in exactly one
   place adds indirection without reducing duplication. Wait until a pattern
   appears in two or more places before extracting.

2. **Barrel files can break tree-shaking.** An `index.ts` that re-exports
   everything from a directory forces the bundler to include all exports even
   when only one is used. Keep barrel files small and specific.

3. **`unknown` is not a drop-in replacement for `any`.** Replacing `any` with
   `unknown` without adding a type guard just moves the error — you still need
   to narrow the type before using it.

4. **Naming conventions are team norms, not universal rules.** If the codebase
   consistently uses a different convention (e.g., `camelCase` filenames), follow
   the existing convention rather than imposing a new one mid-project.

5. **Line count alone is not a quality signal.** A 250-line component with a
   clear single responsibility is better than a 100-line component that imports
   five tightly-coupled helper files. Split based on responsibility, not line count.

6. **`console.error` is sometimes intentional.** Not every console statement is
   dead code — `console.error` in catch blocks provides production diagnostics.
   Only flag `console.log` and `console.debug` for removal.
