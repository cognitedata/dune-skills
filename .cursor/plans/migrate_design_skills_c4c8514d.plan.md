---
name: Migrate Design Skills
overview: Migrate 7 design skills (+ shared resources, /design-review command, and tool-configs) from designstudio into dune-skills-1, adapting to the existing `skills/` directory convention while preserving cross-references. Excludes design-spec. Split across 3 PRs.
todos:
  - id: pr1-foundation
    content: "PR 1 (chore/design-skills-foundation): Copy shared/storybook-links.md, aura-tokens/SKILL.md, aura-components/SKILL.md. Add allowed-tools frontmatter. Update BOTH ci.yml and cd.yml to skip shared/. Backfill 8 missing existing skills + add 2 new design skills to root README table. Create branch, commit, push, open PR."
    status: pending
  - id: pr2-build
    content: "PR 2 (chore/design-skills-build): Copy layout-patterns (+ 9 templates), content-guidelines (+ 11 sub-files), accessibility, error-validation. Add allowed-tools frontmatter. Fix broken ref in cognite-audience.md. Update root README with 4 more skills. Create branch from PR1, commit, push, open PR."
    status: pending
  - id: pr3-review
    content: "PR 3 (chore/design-skills-review): Copy design-quality-checklist, commands/design-review.md, tool-configs (design-review only), design README. Add allowed-tools frontmatter. Rewrite design/README.md (remove design-spec sections, update all paths from .cursor/skills/ to ../skills/design/, rewrite file structure and install instructions, replace all designstudio refs). Rewrite design/tool-configs/README.md (update paths and symlink scripts). Move all 9 design skills under skills/design/ (Option A grouping). Update CI/CD validation loops. Update root README. Grep for stale refs. Create branch from PR2, commit, push, open PR."
    status: pending
isProject: false
---

# Migrate Design Skills into dune-skills-1

## Scope

Migrate 7 design skills + shared resources + /design-review command and adapters from designstudio. **Excludes** design-spec skill, its template, and its tool-configs (cursor/design-spec.mdc, claude-code/design-spec.md).

**Source:** `/Users/sinyeeong/Desktop/designstudio/skills/design/`
**Destination:** `/Users/sinyeeong/dune-skills-1/`

## Decisions Made

- **Overlap with `use-aura-design-system`**: Keep all three alongside each other
- `**allowed-tools` frontmatter**: Add to all 7 incoming skills for consistency
- **Sub-file distribution**: `npx @cognite/dune skills pull` copies entire directories -- no issue
- **Broken reference in content-guidelines**: Clean up during migration
- **PR strategy**: 3 PRs, chore/ branch prefix
- **PR dependency chain**: PR1 -> PR2 -> PR3 (each builds on the previous)

## Cross-Reference Verification

All 55 `../` relative cross-references across the 7 skills were verified. Every one resolves correctly under the new `skills/` parent directory. No changes to cross-references are needed.

The 1 broken reference (`content-guidelines/references/cognite-audience.md` L3 -> `../../../cogdocs/cogdocs-metadata.mdx`) is pre-existing and will be cleaned up in PR2.

## Target Structure (after all 3 PRs)

```
dune-skills-1/
├── skills/
│   ├── # --- Existing (11 skills, unchanged) ---
│   ├── code-quality/SKILL.md
│   ├── use-aura-design-system/SKILL.md
│   ├── ... (9 more existing)
│   │
│   ├── # --- PR 1: Foundation (3 files) ---
│   ├── shared/storybook-links.md
│   ├── aura-components/SKILL.md
│   ├── aura-tokens/SKILL.md
│   │
│   ├── # --- PR 2: Build-time (24 files) ---
│   ├── accessibility/SKILL.md
│   ├── content-guidelines/  (SKILL.md + 11 sub-files)
│   ├── error-validation/SKILL.md
│   ├── layout-patterns/  (SKILL.md + 9 templates)
│   │
│   ├── # --- PR 3: Review (1 file) ---
│   └── design-quality-checklist/SKILL.md
│
├── design/  (PR 3, 5 files)
│   ├── README.md
│   ├── commands/design-review.md
│   └── tool-configs/
│       ├── README.md
│       ├── cursor/design-review.mdc
│       └── claude-code/design-review.md
│
├── README.md
├── .github/workflows/ci.yml
└── .github/workflows/cd.yml
```

---

## PR 1: Foundation -- `chore/design-skills-foundation`

The shared resource and the two "source of truth" skills that everything else depends on.

**Branch from:** `main`

### Files added (3 new)


| Destination                        | Source (relative to designstudio/skills/design/) |
| ---------------------------------- | ------------------------------------------------ |
| `skills/shared/storybook-links.md` | `.cursor/skills/shared/storybook-links.md`       |
| `skills/aura-tokens/SKILL.md`      | `.cursor/skills/aura-tokens/SKILL.md`            |
| `skills/aura-components/SKILL.md`  | `.cursor/skills/aura-components/SKILL.md`        |


### Files edited (3 existing)

`**.github/workflows/ci.yml`** -- add skip for `shared/` in validation loop:

```yaml
for skill in skills/*/; do
  [ "$(basename "$skill")" = "shared" ] && continue
  echo "Validating $skill..."
  npx skills-ref validate "$skill"
done
```

`**.github/workflows/cd.yml**` -- identical change (same validation loop).

`**README.md**` -- backfill all 8 missing existing skills AND add 2 new design skills to Available Skills table. The table currently only has 3 rows; this PR brings it to 13 rows:

New rows to add:

```markdown
| **code-quality** | Reviews Dune apps for code quality, maintainability, and clean code issues |
| **correctness-and-error-handling** | Reviews for bugs, missing error states, unhandled rejections, and edge cases |
| **dm-limits-and-best-practices** | CDF Data Modeling API best practices — concurrency, pagination, batching |
| **integrate-file-viewer** | Integrates CogniteFileViewer to preview CDF files (PDFs, images, text) |
| **performance** | Optimizes Dune apps for speed, render counts, and bundle size |
| **security** | Reviews for security issues — credentials, user input, external data |
| **setup-dune-auth** | Migrates React apps to Dune auth or adds DuneAuthProvider |
| **use-aura-design-system** | Implements or migrates to the Aura Design System in Dune/React apps |
| **aura-tokens** | Enforces semantic Aura design tokens via Tailwind |
| **aura-components** | Guides Aura component selection, prevents custom rebuilds |
```

### Edits to new files (2)

- `skills/aura-tokens/SKILL.md` -- add `allowed-tools: Read, Glob, Grep, Edit, Write` to frontmatter
- `skills/aura-components/SKILL.md` -- add `allowed-tools: Read, Glob, Grep, Edit, Write` to frontmatter

### Why first

Every other design skill has `../shared/storybook-links.md` or `../aura-components/SKILL.md` references. These must exist before anything else lands.

---

## PR 2: Build-time skills -- `chore/design-skills-build`

The 4 skills that activate during development, plus all their templates and sub-files.

**Branch from:** `chore/design-skills-foundation`

### Files added (~24 new)


| Destination                                       | Source (relative to designstudio/skills/design/.cursor/skills/) |
| ------------------------------------------------- | --------------------------------------------------------------- |
| `skills/accessibility/SKILL.md`                   | `accessibility/SKILL.md`                                        |
| `skills/error-validation/SKILL.md`                | `error-validation/SKILL.md`                                     |
| `skills/layout-patterns/SKILL.md`                 | `layout-patterns/SKILL.md`                                      |
| `skills/layout-patterns/templates/` (9 files)     | `layout-patterns/templates/`                                    |
| `skills/content-guidelines/SKILL.md`              | `content-guidelines/SKILL.md`                                   |
| `skills/content-guidelines/docs/` (2 files)       | `content-guidelines/docs/`                                      |
| `skills/content-guidelines/examples/` (1 file)    | `content-guidelines/examples/`                                  |
| `skills/content-guidelines/references/` (5 files) | `content-guidelines/references/`                                |
| `skills/content-guidelines/templates/` (3 files)  | `content-guidelines/templates/`                                 |


**layout-patterns/templates (9):** `sidebar-content.md`, `detail-page.md`, `form-page.md`, `list-page.md`, `settings-page.md`, `split-screen.md`, `three-panel.md`, `full-width-dashboard.md`, `wizard.md`

**content-guidelines sub-files (11):** `docs/claude-figma-integration.md`, `docs/codex-figma-integration.md`, `examples/real-world-improvements.md`, `references/accessibility-guidelines.md`, `references/cognite-audience.md`, `references/content-usability-checklist.md`, `references/patterns-detailed.md`, `references/voice-chart-template.md`, `templates/empty-state-template.md`, `templates/error-message-template.md`, `templates/onboarding-flow-template.md`

### Files edited (1 existing)

`**README.md`** -- add 4 more rows to Available Skills table (bringing total to 17 rows):

```markdown
| **accessibility** | Page-level accessibility — keyboard nav, ARIA, focus management |
| **content-guidelines** | UX writing standards, text patterns, voice and tone |
| **error-validation** | Form validation, loading states, error handling patterns |
| **layout-patterns** | Approved page layouts with responsive specs |
```

### Edits to new files (5)

- `skills/accessibility/SKILL.md` -- add `allowed-tools: Read, Glob, Grep, Edit, Write`
- `skills/error-validation/SKILL.md` -- add `allowed-tools: Read, Glob, Grep, Edit, Write`
- `skills/layout-patterns/SKILL.md` -- add `allowed-tools: Read, Glob, Grep, Edit, Write`
- `skills/content-guidelines/SKILL.md` -- add `allowed-tools: Read, Glob, Grep, Edit, Write`
- `skills/content-guidelines/references/cognite-audience.md` L3 -- remove broken link `[cogdocs/cogdocs-metadata.mdx](../../../cogdocs/cogdocs-metadata.mdx)` and update to note that the canonical source is not available in this repo

---

## PR 3: Design review + skill grouping -- `chore/design-skills-review`

The review orchestrator, command, tool adapters, bundle README, and restructuring all design skills under `skills/design/`.

**Branch from:** `chore/design-skills-build`

### Files added (6 new)

| Destination                                        | Source (relative to designstudio/skills/design/)   |
| -------------------------------------------------- | -------------------------------------------------- |
| `skills/design/design-review/SKILL.md`                  | `.cursor/skills/design-quality-checklist/SKILL.md` (renamed) |
| `skills/design/design-review/workflow.md`               | `commands/design-review.md`                        |
| `skills/design/tool-configs/cursor/design-review.mdc`   | `tool-configs/cursor/design-review.mdc`            |
| `skills/design/tool-configs/claude-code/design-review.md` | `tool-configs/claude-code/design-review.md`      |
| `skills/design/tool-configs/README.md`                  | `tool-configs/README.md`                           |
| `skills/design/README.md`                               | `README.md`                                        |

### Files moved (9 directories -- Option A restructure)

All design skills are grouped under `skills/design/` to separate them from non-design skills:

| From                              | To                                     |
| --------------------------------- | -------------------------------------- |
| `skills/accessibility/`           | `skills/design/accessibility/`         |
| `skills/aura-components/`         | `skills/design/aura-components/`       |
| `skills/aura-tokens/`             | `skills/design/aura-tokens/`           |
| `skills/content-guidelines/`      | `skills/design/content-guidelines/`    |
| `skills/design-quality-checklist/`| `skills/design/design-review/` (renamed) |
| `skills/error-validation/`        | `skills/design/error-validation/`      |
| `skills/layout-patterns/`         | `skills/design/layout-patterns/`       |
| `skills/shared/`                  | `skills/design/shared/`                |
| `skills/use-aura-design-system/`  | `skills/design/use-aura-design-system/`|

Cross-references between design skills (`../X/SKILL.md`) are **unchanged** — skills remain siblings within `skills/design/`.

### Files edited (5 existing)

**`README.md`** -- add 1 final row to Available Skills table (bringing total to all 18 skills), update Contributing section to document `skills/design/<name>/SKILL.md` convention:

```markdown
| **design-review** | Scores app against 10 Aura design criteria with fix suggestions |
```

**`design/README.md`** -- 7 skill links updated to `../skills/design/{name}/SKILL.md`, symlink loop changed to `skills/design/*/`, FAQ link and storybook ref updated, file structure block rewritten to show `skills/design/` grouping.

**`design/tool-configs/README.md`** -- symlink loop changed to `skills/design/*/`.

**`.github/workflows/ci.yml`** -- outer loop skips `design` dir; second loop validates `skills/design/*/` with `shared` skip.

**`.github/workflows/cd.yml`** -- identical change to `ci.yml`.

### Edits to new files -- detailed breakdown

#### `skills/design/design-quality-checklist/SKILL.md`

- Add `allowed-tools: Read, Glob, Grep, Edit, Write` to frontmatter

#### `design/README.md` (substantial rewrite from source)

**Remove entirely:**

- L7-10: "Before development (PRD enrichment)" Quick Start block
- L22-26: "PRD Entry Point" table in Skills at a Glance
- L47-57: Entire "The `/design-spec` Command" section

**Rewrite:**

- L1-3 opening paragraph: Remove design-spec mention, single entry point `/design-review`
- L20: "Two entry points" -> "One entry point" (or similar)
- Skill links: `.cursor/skills/{name}/SKILL.md` -> `../skills/design/{name}/SKILL.md`
- Symlink loop: `.cursor/skills/*/` -> `skills/design/*/`
- `/path/to/designstudio/...` -> `/path/to/dune-skills/...`
- FAQ link: `../skills/design-quality-checklist/SKILL.md` -> `../skills/design/design-quality-checklist/SKILL.md`
- File Structure section: reflects `skills/design/` grouping
- Storybook ref: `../skills/shared/storybook-links.md` -> `../skills/design/shared/storybook-links.md`

#### `design/tool-configs/README.md` (rewrite paths and scripts)

- Symlink script: `skills/*/` -> `skills/design/*/`
- `/path/to/designstudio/...` -> `/path/to/dune-skills/...`

#### Intentionally NOT changed

These files use `.cursor/skills/` paths that are **correct for consumer projects** (where `skills pull` installs skills to `.cursor/skills/`):

- `design/commands/design-review.md` L15, L159: `.cursor/skills/design-quality-checklist/SKILL.md`
- `design/tool-configs/cursor/design-review.mdc` L15, L26, L126-135: skill paths
- `design/tool-configs/claude-code/design-review.md` L10, L67-72: `.cursor/skills/{name}/SKILL.md`

These are distribution artifacts for use in target projects, not references within this repo.

### Final verification

Grep entire repo for:

- `designstudio` -- should find zero
- `skills/design/.cursor` -- should find zero
- `design-spec` in skill files -- should find zero (may appear in design/README.md only if we add a note about exclusion)
- `../skills/aura-\|../skills/accessibility\|../skills/content\|../skills/error\|../skills/layout\|../skills/design-quality\|../skills/shared` in `design/` -- should find zero (all updated to `../skills/design/`)

---

## Resulting Structure (after all 3 PRs)

```
dune-skills/
├── skills/
│   ├── design/                        # All design skills grouped here
│   │   ├── README.md                  # Design skills overview
│   │   ├── tool-configs/              # Per-tool adapter configs
│   │   │   ├── README.md
│   │   │   ├── cursor/design-review.mdc
│   │   │   └── claude-code/design-review.md
│   │   ├── accessibility/SKILL.md
│   │   ├── aura-components/SKILL.md
│   │   ├── aura-tokens/SKILL.md
│   │   ├── content-guidelines/        # SKILL.md + 11 sub-files
│   │   ├── design-review/             # Renamed from design-quality-checklist
│   │   │   ├── SKILL.md
│   │   │   └── workflow.md
│   │   ├── error-validation/SKILL.md
│   │   ├── layout-patterns/           # SKILL.md + 9 templates
│   │   ├── shared/storybook-links.md
│   │   └── use-aura-design-system/SKILL.md
│   ├── code-quality/SKILL.md
│   ├── correctness-and-error-handling/SKILL.md
│   ├── create-client-tool/SKILL.md
│   ├── dm-limits-and-best-practices/SKILL.md
│   ├── integrate-atlas-chat/SKILL.md
│   ├── integrate-file-viewer/SKILL.md
│   ├── performance/SKILL.md
│   ├── security/SKILL.md
│   ├── setup-dune-auth/SKILL.md
│   └── setup-python-tools/SKILL.md
├── README.md
├── .github/workflows/ci.yml
└── .github/workflows/cd.yml
```

## Remaining Edge Cases

1. **Running `/design-review` directly in this repo won't work** -- The workflow and tool-configs reference `.cursor/skills/` paths, which is correct for consumer projects but not this repo. To test in-repo, you'd need to symlink `skills/design/{name}` to `.cursor/skills/{name}`.
2. **Cursor auto-discovery doesn't apply in the new structure** -- Skills live in `skills/design/`, not `.cursor/skills/`. The install instructions in `skills/design/README.md` guide users to symlink via the `skills/design/*/` loop or use `npx @cognite/dune skills pull`.
3. **Tool-configs are Cursor rules, not skills** -- The `.mdc` file goes in a consumer project's `.cursor/rules/` directory. Users follow install instructions in `skills/design/README.md`.

