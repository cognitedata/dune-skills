---
name: pr-splitter
description: >-
  Splits a large committed branch into reviewable GitHub Pull Requests grouped
  by user-facing feature. Each feature receives an independent batch of small
  PRs (~500-700 production lines each, excluding tests) that collectively make
  that feature functional once merged. Encourages isolating UI-only PRs when
  using a ViewModel-style split, with an optional top-of-body design-approval
  banner and screenshot or video placeholders for reviewers. Use when the user
  mentions splitting a PR, breaking up a large branch, making a branch
  reviewable, stacking PRs, a PR being too large, or how to split work—even if
  they do not say PR splitter.
---

# PR Splitter — Feature-Batch Model

Break down a large committed branch into **feature batches** — one batch per
user-facing feature, each batch containing 2-4 small PRs. The goal: a reviewer
can approve a few PRs per day, merge them, and immediately have a functional
slice of the app — even while other feature batches are still pending.

Large PRs tend to get rubber-stamped. Splitting by feature (not by file type)
keeps each batch independently testable and gives each PR a clear narrative.

**The model:**

- **Boilerplate PR**: config, CI/CD, lock files, IDE config. No LOC limit.
- **Shared Foundation (optional)**: types and utilities imported by 2+ batches. ~500-700 lines (excluding tests).
- **Feature Batches**: one batch per feature, branching independently. ~500-700 lines per PR (excluding tests), hard ceiling ~1000.
- **Integration PR (last)**: app entry point, router, global providers.

When all PRs in a feature batch are merged, that feature is independently
testable end-to-end.

**UI vs. non-UI split:** When a feature has meaningful UI, prefer a **dedicated
UI-only PR** (or PRs) after the data/logic layers land. This works best with the
**ViewModel pattern** (or equivalent): prior PRs own state, side effects, and
data shaping; the UI PR is mostly presentational. Use the PR body template in
[reference.md](reference.md#github-pr-body-template) for optional top-of-body
banner and **Visual evidence** placeholders.

---

## Step 0: Prerequisites Check & Project Context Detection

**Prerequisites:**

1. `gh auth status` — confirm the GitHub CLI is authenticated. If not, stop and instruct the user to run `gh auth login`.
2. `git status` — confirm the working tree is clean.
3. Identify the **source branch** (provided by user) and the **ultimate base branch** (e.g., `main`).
4. Confirm the source branch is pushed to remote: `git log --oneline origin/main..HEAD`

**Project context detection (run these and record the results):**

```bash
# Detect repo/project name for branch prefix
git remote get-url origin

# Detect package manager from lock files
git diff --name-only main HEAD | rg "(pnpm-lock|yarn\.lock|package-lock)"

# Detect CI/CD pipelines
git diff --name-only main HEAD | rg "\.github/workflows/"
```

From these results:

- **Branch prefix**: Derive a short prefix from the repo or project name (e.g., `work-package-generator` → `wpg`, `my-api-service` → `api`, `operations-summarizer` → `ops-sum`). Use this prefix for all branch names: `<prefix>/pr1-boilerplate`.
- **Package manager**: `pnpm-lock.yaml` → use `pnpm`; `yarn.lock` → use `yarn`; `package-lock.json` → use `npm`; none found → use `npm` as default.
- **Has deployment pipeline**: If any workflow file matching `cd.yml`, `deploy.yml`, `release.yml`, or similar exists in the diff, set `HAS_CD=true`. This affects PR descriptions (see Step 5).

---

## Step 1: Analyze the Full Diff & Identify Features

### 1a — Get the full file list and line counts

```bash
git diff --stat main HEAD
```

Build an annotated file list with the **counted line total** for each file. Counting rules:

- Lock files, generated files (`*.generated.*`), binaries, and docs are excluded from LOC counts
- Test files (`*.test.*`, `*.spec.*`, `__tests__/`, `*.test-utils.*`) are excluded from LOC counts — they are tracked separately but never count against a PR's LOC budget
- Only additions count (not deletions or whitespace-only changes)
- Lock files always go in the boilerplate PR regardless of size

### 1b — Identify user-facing features

After gathering the file list, read the directory structure to identify distinct user-facing features. Look at:

```bash
# Identify feature areas from pages, features, and component directories
git diff --name-only main HEAD | rg "src/(pages|features|components)/"
```

Group what you find into named features. For example:

- `src/pages/SummaryPage.tsx`, `src/pages/SummaryDetailModal.tsx` → **"Summary" feature**
- `src/pages/TestingPage.tsx`, `src/pages/TestHistoryPage.tsx` → **"Testing" feature**
- `src/pages/Dashboard.tsx` → **"Dashboard" feature**
- `src/features/dashboard-agent/` → part of **"Dashboard" feature**

Record each feature name — these become your batch labels. If a file is shared across multiple features (e.g., a global context, shared hook), it goes in the **Shared Foundation** layer.

---

## Step 2: Group Files into Batches

### Boilerplate PR (PR 1 — no LOC limit)

Place all of the following in PR 1, regardless of total size:

- CI/CD workflows (`.github/workflows/`), build configs (`vite.config.ts`, `tsconfig.json`)
- Package manifests and lock files (`package.json`, `pnpm-lock.yaml`)
- Linter/formatter configs (`biome.json`, `.eslintrc`, `prettier.config.js`)
- Environment templates, container/infra files, IDE configs
- Repository metadata (`.gitignore`, `CODEOWNERS`, `components.json`)
- Test runner config, README/docs, CSS/Tailwind config, `index.html`

The PR description for this batch explicitly tells reviewers which files deserve attention and instructs them to skip the rest.

This PR should make the repo independently buildable and pass CI on its own.

---

### Shared Foundation PR (optional)

Create this PR only if **two or more feature batches** import the same types, utilities, or services. If every type and utility is used by exactly one feature, skip this PR and include those files directly in the relevant feature batch instead.

Files that belong here:

- Global TypeScript type definitions consumed by multiple features
- Pure utility functions with no feature-specific logic, used across features (e.g., `array.ts`, `chunking.ts`, `errorHandling.ts`)
- Shared service clients or SDK wrappers used by multiple features (e.g., a CDF client helper)
- Global CSS / design tokens / theme files used by all UI components
- Shared UI primitives (shadcn/radix components, base button, input, dialog, etc.)
- Global state context providers that coordinate multiple features

Target: ~500-700 counted lines (excluding tests). If the shared layer exceeds 700 lines, sub-split it:

- PR 2a: shared types + pure utility functions
- PR 2b: shared UI primitives + styling
- PR 2c: shared service/API layer + context providers

---

### Feature Batches

For each user-facing feature identified in Step 1b, create a batch. Each batch contains **2-4 PRs** following this structure (skip layers that have no files):

**Batch PR A — Feature Types & Data Layer**

- Feature-specific TypeScript types and interfaces
- CDF query hooks, API calls, data-fetching functions for this feature
- Query key constants for this feature

Target: ~500-700 counted lines (excluding tests).

**Batch PR B — Feature Logic & Utilities**

- Business logic utilities specific to this feature (generation, formatting, calculation)
- Feature-specific state hooks (orchestration hooks that combine data + logic)
- PDF/export utilities if feature-specific

Target: ~500-700 counted lines (excluding tests). If this layer exceeds 700, split along natural boundaries:

- e.g., data-transformation utils vs. generation/output utils
- e.g., one hook per file if hooks are large

**Batch PR C — Feature UI & Page**

- Feature-specific components (dialogs, panels, cards)
- The page(s) that compose this feature
- Feature module entry points (`index.ts`)

Target: ~500-700 counted lines (excluding tests). If page files are large, split by component:

- e.g., one PR per large page component
- e.g., shared feature components (dialogs, result panels) in one PR, the main page in another

**Prefer isolating “UI-only” changes** in PR C (or a follow-up PR C2) when logic and ViewModels already merged in A/B: reviewers can approve visual work faster when the diff is clearly presentational. Use the **UI PR banner** and **Visual evidence** sections from [reference.md](reference.md#github-pr-body-template).

**Batch PR D — Feature Tests (optional)**

- Unit and integration tests for this feature
- Tests never count toward the LOC budget of any PR — include them in the most relevant A/B/C PR unless they are large enough to impair review focus, in which case create a dedicated test PR

**Completeness requirement**: when every PR in the batch has been merged into `main`, a developer must be able to run the app and exercise that feature end-to-end. If the feature depends on the integration PR to be accessible via navigation, state this explicitly in the final batch PR description.

---

### LOC Discipline

1. **Tests are excluded.** Test files (`*.test.*`, `*.spec.*`, `__tests__/`) never count toward a PR's LOC budget. Count only production code.
2. **Target**: ~500-700 counted lines per feature PR (production code only).
3. **If oversize**: look for natural sub-splits within the layer (separate sub-components, split utils by concern).
4. **Exception ceiling**: ~900 lines — if exceeded, the PR description must include the exact count, per-file breakdown, and a concrete reason why further splitting would break compilation or review.
5. **Hard ceiling**: ~1000 lines. Restructure the batch rather than accept a PR this large.
6. **Boilerplate PR is exempt.** Document its contents clearly instead.

---

### Integration PR (last)

The final PR wires all features together:

- App root component (`App.tsx`)
- Main entry point (`main.tsx`)
- Router configuration
- Top-level navigation component (if shared across features)
- Global context providers that span multiple features
- Any remaining shared utilities not captured earlier

Target: ~500-700 counted lines (excluding tests). Accept an exception here if the app's entry point file is inherently large.

---

### CI/CD Ordering Principle

The dependency order is:

```
Boilerplate → Shared Foundation → Feature Batches (independent of each other) → Integration
```

- **Between batches**: feature batches are independent. Batch B does not need to wait for Batch A to merge — they both branch from the same base. Different reviewers can review different batches in parallel.
- **Within a batch**: PRs stack in order (A → B → C → D). PR B branches from PR A because B's logic depends on A's types.
- **Integration PR**: always last; branches from the final merged state of `main` after all batches are in.
- **CD pipeline**: if `HAS_CD=true`, the deployment pipeline must only trigger on merges to `main`. State this in every PR description.

---

## Step 3: Present and Confirm the Strategy

Before executing, present the full proposed plan to the user. Use this table format:

| PR | Branch Name | Base Branch | Batch | Layer | Counted Lines | Functional When? |
|----|-------------|-------------|-------|-------|---------------|-----------------|
| 1 | `<prefix>/pr1-boilerplate` | `main` | Boilerplate | Config, CI/CD, lock files, IDE | uncapped | — |
| 2 | `<prefix>/pr2-shared-types` | `pr1-boilerplate` | Shared Foundation | Global types + pure utils | ~XXX | — |
| 3 | `<prefix>/pr3-shared-ui` | `pr2-shared-types` | Shared Foundation | UI primitives + global styles | ~XXX | — |
| 4 | `<prefix>/pr4-summary-data` | `pr3-shared-ui` | Summary Batch | Feature types + CDF hooks | ~XXX | — |
| 5 | `<prefix>/pr5-summary-logic` | `pr4-summary-data` | Summary Batch | Summary generation + formatting utils | ~XXX | — |
| 6 | `<prefix>/pr6-summary-ui` | `pr5-summary-logic` | Summary Batch | Summary components + page | ~XXX | **Summary feature functional** |
| 7 | `<prefix>/pr7-testing-data` | `pr3-shared-ui` | Testing Batch | Feature types + CDF hooks | ~XXX | — |
| 8 | `<prefix>/pr8-testing-logic` | `pr7-testing-data` | Testing Batch | Testing utils + state hooks | ~XXX | — |
| 9 | `<prefix>/pr9-testing-ui` | `pr8-testing-logic` | Testing Batch | Testing components + pages | ~XXX | **Testing feature functional** |
| 10 | `<prefix>/pr10-integration` | `main` | Integration | App.tsx, main.tsx, router | ~XXX | **Full app functional** |

**Base Branch** = the last shared foundation PR (or boilerplate) for a batch's first PR, then the previous PR within the batch. **Functional When?** = blank except for the final PR in each batch. Note which batches are independent. Also state: detected package manager, branch prefix, `HAS_CD=true/false`.

**Wait for explicit user approval before executing.**

---

## Step 4: Execute the Split

The source branch is never modified. New branches copy paths from it with `git checkout <source-branch> -- <path>…`, then commit and push.

**Concrete bash patterns** (boilerplate from `main`, first PR in a batch, stacked PRs within a batch, parallel batch from shared foundation) and the **key rule** (never base one feature batch on another) are in [reference.md](reference.md#cherry-pick-patterns).

---

## Step 5: Create GitHub Pull Requests

Before writing each PR description, **read the key files in that PR**. Generic labels ("data layer") are not enough — describe what the code does in this application, naming key functions and components.

**Template rule:** `<CONDITIONAL>…</CONDITIONAL>` blocks are authoring metadata only. Do **not** publish those wrapper lines or their instructional sentence to GitHub; include only the inner markdown when the condition applies (optional UI banner at the very top, or the LOC note bullet block).

The full `gh pr create` example, PR body template (through Notes to reviewer), boilerplate adjustments, and UI-only banner guidance are in [reference.md](reference.md#github-pr-body-template).

After each `gh pr create` succeeds, print the returned PR URL for the user.

---

## Step 6: Final Verification

After all PRs are created:

1. Run `gh pr list` to confirm all PRs are open with the correct base branches.

2. Verify the following:

   - Every feature batch has exactly one PR with a non-empty "Functional When?" entry (the final batch PR).
   - Each independent batch's first PR correctly bases off the shared foundation (or boilerplate) — not off another feature batch.
   - The integration PR bases off `main` (it will be rebased after all batches merge).

3. Remind the user:

   - **Parallel review**: feature batches are independent — assign different reviewers to different batches.
   - **Merge order within a batch**: PRs in a batch merge in sequence (A → B → C). GitHub auto-updates the next PR's diff after each merge.
   - **Merge order between batches**: any order, but all batches merge before the integration PR.
   - **UI PRs**: fill **Visual evidence** (screenshots; optional Loom/GIF for flows). Use the **UI-only banner** at the top when the PR is presentational and designs are pre-approved. Isolating UI in its own PR (after ViewModel/logic PRs) speeds up approval.
   - Add JIRA ticket links for all PRs as appropriate.
   - Run lint, format check, and tests before requesting review.
   - Self-review each PR on GitHub before marking ready for peer review.

4. **CI/CD reminder**: If `HAS_CD=true`, confirm the deployment workflow triggers only on `main` pushes. Check the `on:` trigger in the CD workflow file. If it triggers on all branch pushes, warn the team that each batch PR merge may attempt a deploy of an incomplete app.

---

## Step 7: Post-Split Patch Workflow

When fixing stacked branches after review, shared-foundation cascades, or re-splitting a wide change, follow [reference.md](reference.md#post-split-patch-workflow).

---

## Gotchas

1. **Never base a feature batch on another feature batch.** Both batches branch
   from the shared foundation (or boilerplate). Basing batch B on batch A creates
   a serial dependency that defeats parallel review.

2. **`git checkout <source> -- <file>` overwrites without merging.** If you've
   already modified a file in the target branch, the checkout replaces it entirely.
   Use `git diff` to verify before checking out files.

3. **Lock files can cause merge conflicts between batches.** If multiple batches
   modify `package.json` (e.g., adding different dependencies), the lock file
   will conflict when the second batch merges. Consolidate all dependency changes
   in the boilerplate PR to avoid this.

4. **Rebasing changes commit hashes.** After `git rebase` + `git push --force-with-lease`,
   existing review comments on GitHub may become outdated. Warn reviewers before
   force-pushing a branch they've already reviewed.

5. **The integration PR should be created last.** It needs to be rebased onto
   `main` after all feature batches have merged, so creating it early just means
   more rebase work later.

---

## Additional resources

- [reference.md](reference.md) — cherry-pick bash patterns, full GitHub PR body template (`gh pr create`), boilerplate and UI-only notes, post-split rebasing workflow.
