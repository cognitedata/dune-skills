---
name: count-real-code-loc
description: Count lines of real project-authored code, excluding template, generated, and third-party code. Use when the user asks for line count, LOC, code size, how much code they've written, or wants to understand the scope of a codebase — even if they just say "how big is this app?"
---

# Count Real Code (LOC)

Count only project-authored source code. Template/registry components (e.g.
shadcn, Aura UI in `src/components/ui`), generated output, and dependencies
inflate the count and misrepresent how much code the team actually maintains.

## What to Exclude

| Category | Typical paths |
|----------|--------------|
| Build output & deps | `node_modules`, `.git`, `dist`, `build`, `coverage`, `.next`, `out` |
| Template/registry UI | `src/components/ui` (shadcn/Aura registry components) |
| Generated code | Lock files, auto-generated types, GraphQL codegen output |
| Config files | `vite.config.ts`, `tailwind.config.ts`, `tsconfig.json` |

Scan the repo before counting — look for any directory that contains
copy-pasted or CLI-generated code that the team doesn't actively maintain.

## How to Count

Use `rg --files` to list source files, then pipe to `wc -l` for line counts.
This works cross-platform (Windows, macOS, Linux).

**Option A — Include only app-code directories (recommended)**

```bash
rg --files --type ts src/ --glob "!src/components/ui/**" | xargs wc -l
```

**Option B — All of `src/` minus excluded paths**

```bash
rg --files --type ts src/ --glob "!src/components/ui/**" --glob "!**/*.generated.*" --glob "!**/*.d.ts" | xargs wc -l
```

**Per-directory breakdown**

```bash
for dir in src/components src/hooks src/contexts src/utils src/types src/stores src/pages; do
  count=$(rg --files --type ts "$dir" --glob "!src/components/ui/**" 2>/dev/null | xargs cat 2>/dev/null | wc -l)
  echo "$count $dir"
done
```

## Customizing

- **Include more dirs**: Add paths to the `rg --files` command or the loop
- **Exclude more paths**: Add `--glob "!path/to/exclude/**"` flags
- **Other extensions**: Add `--glob "*.css"` or use `--type js` for JS files

## Report

Use this format:

### Lines of Code — [App Name]

| Directory | Lines |
|-----------|-------|
| `src/components` (excl. `ui/`) | N |
| `src/hooks` | N |
| `src/utils` | N |
| ... | ... |
| **Total** | **N** |

*Excluded: `src/components/ui` (registry), `node_modules`, generated files.*

## Gotchas

1. **Barrel files inflate counts.** An `index.ts` that re-exports 20 components
   adds lines that aren't real logic. Note if a significant portion of the count
   is barrel files.

2. **Test files may or may not count.** Ask the user if they want `*.test.ts` /
   `*.spec.ts` included. By default, include them — tests are real code the team
   maintains.

3. **Auto-generated type files look like real code.** Files like `*.generated.ts`
   or GraphQL codegen output can be hundreds of lines. Exclude them with
   `--glob "!**/*.generated.*"`.

4. **`wc -l` counts blank lines.** If the user wants non-blank lines only, pipe
   through `rg -c "." <file>` instead of `wc -l`. This usually reduces the count
   by 15-25%.
