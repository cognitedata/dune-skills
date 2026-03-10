---
name: count-real-code-loc
description: Count lines of "real" project code by excluding template, generated, and third-party code (e.g. UI registry, node_modules). Use when the user asks for line count, LOC, "real" code size, or code that excludes templates/shadcn/Aura components.
---

# Count Real Code (LOC)

"Real" code means project-authored source only. Exclude code that was included from a template or registry (e.g. shadcn/Aura in `src/components/ui`), generated output, and dependencies.

## What to Exclude

- **Build/deps**: `node_modules`, `.git`, `dist`, `build`, `coverage`, `.next`, `out`
- **Template/registry UI**: Paths that contain UI components added by a template or CLI (e.g. shadcn, Aura). In this repo that is `src/components/ui`.

## How to Count

From the **app root** (e.g. `apps/guided-rca`):

**Option A – Include only specific dirs (recommended)**  
Count only the directories that contain real app code. Omit `src/components/ui` when including `src/components`.

```bash
find src/components src/contexts src/controllers src/hooks src/stores src/utils src/types \
  -type f \( -name "*.ts" -o -name "*.tsx" \) \
  2>/dev/null | grep -v 'src/components/ui' | xargs wc -l 2>/dev/null | tail -1
```

**Option B – Whole `src` minus template paths**  
Count all of `src` except known template/registry subtrees:

```bash
find src -type f \( -name "*.ts" -o -name "*.tsx" \) \
  -not -path "*/node_modules/*" \
  -not -path "*/.git/*" \
  2>/dev/null | grep -v 'src/components/ui' | xargs wc -l 2>/dev/null | tail -1
```

**Per-directory breakdown** (Option A style):

```bash
find src/components src/contexts src/controllers src/hooks src/stores src/utils src/types \
  -type f \( -name "*.ts" -o -name "*.tsx" \) \
  2>/dev/null | grep -v 'src/components/ui' | xargs wc -l 2>/dev/null
```

## Customizing Includes/Excludes

- **Include more dirs**: Add paths to the `find` list (e.g. `src/commands`, `src/data`, `src/AtlasAIChat`).
- **Exclude more template paths**: Add more `grep -v 'path/to/exclude'` (paths are relative; no leading slash).
- **Other extensions**: Add `-o -name "*.js"` etc. in the `find` for JS/CSS if needed.

Report the total and, if useful, a short table by directory.
