---
name: Fix design skills PRs
overview: "Fix PR #33 in place on chore/design-skills-review branch (retarget to main, close #31/#32). Adds design- prefix, makes skills self-contained, merges use-aura-design-system into design-aura-components, and fixes CI."
todos:
  - id: retarget-pr
    content: "Retarget PR #33 (chore/design-skills-review) to main, close PRs #31 and #32 as superseded"
    status: pending
  - id: prefix-and-flatten
    content: Rename and move all design skills to flat skills/design-<name>/ directories with design- prefix
    status: pending
  - id: merge-aura-skills
    content: Merge use-aura-design-system content into design-aura-components, leave a deprecation stub at use-aura-design-system
    status: pending
  - id: inline-storybook-links
    content: Create per-skill storybook-links.md with relevant subsets, delete skills/design/shared/
    status: pending
  - id: fix-cross-refs
    content: Replace all 56 ../skill-name/SKILL.md relative paths with skill-name-only references using design- prefixed names
    status: pending
  - id: fix-tool-configs
    content: Move tool-configs into design-review skill dir, replace hardcoded .cursor/skills/ paths with skill-name references
    status: pending
  - id: fix-ci-validation
    content: Replace two-loop skip-list CI with single find-based SKILL.md auto-discovery in ci.yml and cd.yml
    status: pending
  - id: trim-large-skills
    content: Extract cross-reference and what-to-check blocks from design-review criteria into sub-files, keep rubrics in SKILL.md (~340 lines)
    status: pending
  - id: style-alignment
    content: Convert XML role tags to markdown sections, add contributing style note to README
    status: pending
  - id: update-readme
    content: Update README skills table and contributing section with new design- prefixed names
    status: pending
isProject: false
---

# Fix Design Skills PRs (#31, #32, #33)

## Strategy

Work directly on the existing `chore/design-skills-review` branch (PR #33), which already contains all files from PRs #31, #32, and #33. Retarget PR #33 to merge into `main` (currently targets `chore/design-skills-build`). Close PRs #31 and #32 as superseded with a comment linking to #33. Apply all fixes as new commits on this branch -- the messy intermediate history gets squash-merged when the PR lands on `main`.

## 1. Flatten with `design-` prefix

**Problem**: `skills/design/*/SKILL.md` nesting breaks the flat `skills/<name>/SKILL.md` convention, complicates CI with skip-lists, and has unverified CLI path resolution.

**Fix**: All design skills live flat at `skills/design-<name>/SKILL.md`. The `design-` prefix provides grouping without nesting. Final skill list:


| Directory                           | Source                                                                                                     |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `skills/design-aura-tokens/`        | from `skills/design/aura-tokens/`                                                                          |
| `skills/design-aura-components/`    | from `skills/design/aura-components/` + merged `use-aura-design-system` content                            |
| `skills/design-accessibility/`      | from `skills/design/accessibility/`                                                                        |
| `skills/design-content-guidelines/` | from `skills/design/content-guidelines/` (with sub-dirs `docs/`, `references/`, `templates/`, `examples/`) |
| `skills/design-error-validation/`   | from `skills/design/error-validation/`                                                                     |
| `skills/design-layout-patterns/`    | from `skills/design/layout-patterns/` (with `templates/` sub-dir)                                          |
| `skills/design-review/`             | from `skills/design/design-review/` (with `workflow.md`, `tool-configs/`)                                  |
| `skills/use-aura-design-system/`    | kept as **deprecation stub** (see section 2)                                                               |


Update the `name:` field in every SKILL.md frontmatter to match the new directory name (e.g., `name: design-aura-tokens`).

Delete `skills/design/` entirely after moving.

## 2. Merge `use-aura-design-system` into `design-aura-components`

**Problem**: `use-aura-design-system` (56 lines) overlaps with the new `aura-tokens` and `aura-components` skills but references neither. Its unique content -- shadcn replacement rules, npm export verification, setup checklist, browser auth -- is fundamentally about component migration.

**Fix (Option A -- merge + deprecation stub)**:

**Merge into `design-aura-components`:**

- **Setup checklist** (read package.json, entry file, target component) becomes a preamble section before the decision tree
- **Replacement rule** (verify npm exports, wrapper swapping, keep local if no Aura equivalent) becomes a new usage principle in `<usage-principles>`
- **Reference URLs** (primitives docs, patterns docs, npm) complement the existing Storybook links
- **Browser auth** note folds into the existing `<visual-checks>` section

Net addition: ~30 lines (some content already covered by `aura-components`).

**Deprecation stub at `[skills/use-aura-design-system/SKILL.md](skills/use-aura-design-system/SKILL.md)`:**

```markdown
---
name: use-aura-design-system
description: "Deprecated — merged into design-aura-components. Use that skill instead."
allowed-tools: Read
---

# Deprecated

This skill has been merged into **design-aura-components**.

For component selection and implementation, see the **design-aura-components** skill.
For styling and token rules, see the **design-aura-tokens** skill.
```

This keeps CLI compatibility (`--skill use-aura-design-system` still resolves) while redirecting the agent immediately.

## 3. Make skills self-contained (fix cross-references)

**Problem**: 56 relative `../` cross-references across 8 skills break after flat install to `.cursor/skills/<name>/`. The `shared/` directory (no SKILL.md) never gets pulled by the CLI.

### 3a. Inline storybook links

Each skill gets its own `storybook-links.md` containing only the subset it needs:


| Skill                     | Subset                                                            |
| ------------------------- | ----------------------------------------------------------------- |
| `design-aura-tokens`      | Foundations only (Colors, Effects, Layout, Typography) -- 4 links |
| `design-aura-components`  | All component sections + AI components -- full list               |
| `design-review`           | Full list (foundations + all components)                          |
| `design-error-validation` | Form support + feedback/overlays -- ~12 links                     |
| `design-layout-patterns`  | Foundations > Layout -- 1 link                                    |


Reference via `./storybook-links.md` (same-directory, always resolves). Delete `skills/design/shared/` entirely.

### 3b. Replace relative paths with skill-name references

All 56 `../skill-name/SKILL.md` references become install-path-agnostic skill-name references:

```markdown
<!-- Before -->
See `../content-guidelines/SKILL.md` for message patterns.

<!-- After -->
See the **design-content-guidelines** skill for message patterns.
```

Note: skill names now use the `design-` prefix. The one sub-file cross-reference (`../../accessibility/SKILL.md` in `content-guidelines/references/accessibility-guidelines.md`) also gets this treatment.

**Sub-file cross-reference** (`[design-review/SKILL.md](skills/design/design-review/SKILL.md)` line 500):
`../content-guidelines/references/accessibility-guidelines.md` references a sub-file of another skill. Can't use a simple skill-name reference for sub-files. Change to: "See also the accessibility guidelines reference in the **design-content-guidelines** skill." The agent loads the skill directory and discovers the sub-file itself.

**Deduplicate InputGroup sizing**: currently duplicated in both `aura-tokens` and `aura-components`. Keep the full rule with examples in `design-aura-components` only; have `design-aura-tokens` reference it via skill-name.

## 4. Fix tool-configs and workflow paths

**Problem**: `tool-configs/` hardcodes `.cursor/skills/design-review/SKILL.md` (and 5 other skill paths) in 5 files, 13 locations. Additionally, the cursor rule has a pre-existing bug and all tool-config files reference skills by old unprefixed names.

**Fix**: Move `skills/design/tool-configs/` into `skills/design-review/tool-configs/`. Apply these path fixes:

**Same-skill references** (design-review's own files -- keep `.cursor/skills/` form since tool-configs are installed to `.cursor/rules/` or `.claude/commands/`, not inside the skill dir):

- `.cursor/skills/design-review/SKILL.md` -- keep as-is (correct install path, skill name unchanged)
- `.cursor/skills/design-review/workflow.md` -- keep as-is

**Pre-existing bug in cursor rule** (`[design-review.mdc](skills/design/tool-configs/cursor/design-review.mdc)` line 25):

- `design-review/SKILL.md` (ambiguous, resolves to project root) -> `.cursor/skills/design-review/SKILL.md`

**Cross-skill references** (use skill-name references with `design-` prefix):

- `.cursor/skills/aura-tokens/SKILL.md` -> "the **design-aura-tokens** skill"
- Same for all 6 cross-skill paths in claude-code command (lines 67-72)

**Cursor rule skill list** (`[design-review.mdc](skills/design/tool-configs/cursor/design-review.mdc)` lines 15-21):

- Update Setup section to list `design-` prefixed names

**Cursor rule cross-referencing section** (`[design-review.mdc](skills/design/tool-configs/cursor/design-review.mdc)` lines 126-135):

- `Read aura-tokens/SKILL.md` -> "Read the **design-aura-tokens** skill"
- Same for all 10 criterion skill references

**tool-configs/README.md symlink scripts**:

- Update from `skills/design/*/` with skip-list to `skills/design-*/` (no skip-list)

Files affected:

- `[tool-configs/cursor/design-review.mdc](skills/design/tool-configs/cursor/design-review.mdc)` (bug fix + skill list + cross-ref section + 2 path refs)
- `[tool-configs/claude-code/design-review.md](skills/design/tool-configs/claude-code/design-review.md)` (8 path refs)
- `[design-review/workflow.md](skills/design/design-review/workflow.md)` (2 same-skill refs -- keep as-is)
- `[tool-configs/README.md](skills/design/tool-configs/README.md)` (symlink script + 2 path refs)

## 5. Robust CI validation

**Problem**: 4 manual skip-list entries across 2 loops in both `ci.yml` and `cd.yml`.

**Fix**: With a flat structure, the original single loop works again with no exclusions. But to be future-proof, use `find`-based auto-discovery:

```yaml
- name: Validate skills
  run: |
    find skills -maxdepth 2 -name 'SKILL.md' -print0 | while IFS= read -r -d '' skill; do
      dir="$(dirname "$skill")"
      echo "Validating $dir..."
      npx skills-ref validate "$dir"
    done
```

Only directories containing a `SKILL.md` get validated. Zero manual exclusions needed now or in the future.

Apply to both `[.github/workflows/ci.yml](.github/workflows/ci.yml)` and `[.github/workflows/cd.yml](.github/workflows/cd.yml)`.

## 6. Trim large skill files

**Problem**: `design-review/SKILL.md` (720 lines) and `content-guidelines/SKILL.md` (459 lines) are 3-6x larger than the repo median (~200 lines).

**Fix**:

- `**design-review/SKILL.md`**: Keep in SKILL.md: role, finding-types, when-to-reference, scoring-instructions, **question + rubric table for each criterion** (~8 lines each = 80 lines for all 10), scoring-summary, auto-fix-rules, pattern-grouping, fix-output-format, output-format. This totals ~340 lines. Extract only the verbose `<cross-reference>` and `<what-to-check>` blocks from each criterion into `design-review/criteria/` sub-files (e.g., `01-aura-tokens.md`, ..., `10-accessibility.md`). The SKILL.md references them: "For criterion 1 check details, read `./criteria/01-aura-tokens.md`."
**Why keep rubrics in SKILL.md**: The workflow.md and cursor rule both tell the agent to "use the quantitative rubric thresholds from the design review skill" for scoring. If rubrics move to sub-files, the agent needs 11 file reads instead of 1. Keeping rubrics inline means the agent reads SKILL.md once and can score all 10 criteria. Sub-files are read only when actively investigating a specific criterion.
- `**content-guidelines/SKILL.md`**: Already has sub-files in `docs/`, `references/`, `templates/`. Review whether the 459-line SKILL.md duplicates content from those sub-files and extract where it does.

## 7. Authoring style alignment (soft)

**Problem**: Existing skills use `## Step N -- Title`. New design skills use XML tags (`<role>`, `<when-to-reference>`, `<instruction>`, etc.). 38 distinct XML tag types in `design-review` alone.

**Fix**: Not a blocker -- XML tags are valid LLM prompting. For consistency:

- Convert `<role>` blocks to `## Role` markdown sections across all design skills
- Keep XML tags where they serve a machine-parsing purpose (e.g., `<rubric>` scoring blocks, `<example>` type annotations)
- Add a style note to the README Contributing section

## 8. Update README and delete stale docs

Update [README.md](README.md):

- Skills table: rename all design skills with `design-` prefix, remove `use-aura-design-system` row (or mark deprecated), add note about `design-aura-components` absorbing it
- Contributing section: remove the `skills/design/<name>/SKILL.md` path, keep only `skills/<name>/SKILL.md`
- Note the `design-` prefix convention for design skills

**Delete `[skills/design/README.md](skills/design/README.md)`** (280 lines): This comprehensive README documents the old nested `skills/design/` structure, including installation instructions, file structure tree, FAQ, and mermaid diagrams -- all referencing the old paths. After flattening, it becomes stale. The root `README.md` covers installation; the `design-review` skill is self-documenting via SKILL.md + workflow.md. No replacement needed.

## Execution notes

**Criteria sub-files need cross-reference conversion too**: When extracting `<cross-reference>` and `<what-to-check>` blocks from `design-review/SKILL.md` into `criteria/` sub-files, those blocks contain `../skill-name/SKILL.md` references that must also be converted to `design-` prefixed skill-name references. Do not extract verbatim.

`**workflow.md` needs zero edits**: Its 2 `.cursor/skills/design-review/` paths are same-skill and correct. The tool-configs edits are in the `.mdc`, `.md`, and `README.md` files only.

**Frontmatter `description:` fields are fine**: Checked all 7 design skill descriptions -- none reference other skills by name in the `description:` field. Only `name:` needs updating.

## Summary of file operations (87 total references)


| Action        | Details                                                                            |
| ------------- | ---------------------------------------------------------------------------------- |
| Move + rename | `skills/design/aura-tokens/` -> `skills/design-aura-tokens/` (and 6 more)          |
| Move          | `skills/design/tool-configs/` -> `skills/design-review/tool-configs/`              |
| Merge         | `use-aura-design-system` content into `design-aura-components/SKILL.md`            |
| Replace       | `use-aura-design-system/SKILL.md` with deprecation stub                            |
| Create        | 5x per-skill `storybook-links.md` files                                            |
| Create        | 10x `design-review/criteria/*.md` sub-files (with cross-refs already converted)    |
| Edit          | 59 `../` cross-references across design skills and 1 sub-file (-> skill-name refs) |
| Edit          | 7 `.cursor/skills/` cross-skill paths in tool-configs (-> skill-name refs)         |
| Edit          | 16 bare old skill names in cursor rule (6 in setup list + 10 in cross-ref section) |
| Fix           | 1 pre-existing bug in cursor rule line 25 (ambiguous path)                         |
| Keep          | 4 `.cursor/skills/design-review/` same-skill paths (workflow.md + tool-configs)    |
| Edit          | 1 symlink script in tool-configs/README.md                                         |
| Edit          | `.github/workflows/ci.yml` and `cd.yml`                                            |
| Edit          | `README.md` (skills table + contributing section)                                  |
| Edit          | 7 SKILL.md frontmatter `name:` fields                                              |
| Delete        | `skills/design/` directory entirely (including `skills/design/README.md`)          |


