# /design-review Command

Tool-agnostic command definition for running a design quality
review. Tool-specific adapters (Cursor rule, Claude Code command)
reference this file for shared logic.

## Trigger

Builder types `/design-review` in their tool.

## Workflow

### Step 1: Load the design review skill

Read `.cursor/skills/design-review/SKILL.md` to load all
10 criteria with rubrics and cross-references.

### Step 2: Check for prior reviews

Look for `.design-reviews/history.json` in the project root.
If it exists, load previous scores for comparison.
If it does not exist, this is the first review.

### Step 3: Scan the codebase

For each criterion, perform the appropriate verification:

**CODE/GREP verifiable (criteria 1, 2, 3, 4, 6, 8):**

Criterion 1 — Aura token consistency:
- grep for hardcoded hex values: `style={{`, `#[0-9a-fA-F]{3,8}`
- grep for Tailwind defaults: `text-gray-`, `bg-red-`, `bg-green-`,
  `bg-blue-`, `text-black`, `bg-white`, `border-gray-`
- grep for inline style overrides on Aura components
- Count violations per file
- Search for imports from `@/components/ui/` that are NOT from
  `@/components/ui/aura/` — compare against the Aura component
  inventory. Flag non-Aura imports when an Aura equivalent exists.
- Search for `var(--[a-z]+-[0-9]+)` in .tsx files to find raw
  color ramp variable usage in style attributes
- Search for `var(--color-` to find misnamed CSS variables

Criterion 2 — Navigation & hierarchy:
- Check for heading hierarchy (h1, h2, h3 usage)
- Check for Breadcrumb, Sidebar, Segmented Control imports
- Verify one h1 per page component
- Check for panel collapsibility in multi-panel layouts
  (toggle buttons, collapse state management)
- Verify content-heavy views maximize primary content area
- Check dense data views (charts, timeseries, documents) for
  fullscreen or wide-view expansion capability

Criterion 3 — Labels & language:
- grep for generic button labels: `>Submit<`, `>OK<`, `>Click here<`,
  `>Yes<`, `>No<`, `>Confirm<`, `>Done<`, `>Go<`, `>Proceed<`
- Check for placeholder-only inputs (no Label component)
- Check for Title Case in headings

Criterion 4 — Feedback & validation:
- grep for loading/skeleton usage
- Check for error handling patterns (try/catch with toast)
- Check for form validation patterns
- grep for generic error messages: `"Error"`, `"Something went wrong"`

Criterion 6 — Error prevention:
- grep for delete/remove handlers without Dialog/AlertDialog
- Check for confirmation patterns before destructive actions
- grep for `"Yes"` / `"No"` in Dialog buttons

Criterion 8 — Empty states:
- Check data-displaying components (Table, List, DataTable)
  for empty state handling
- grep for `.length === 0` or `.length > 0` patterns
- Verify empty states have CTA buttons

**VISUAL/MANUAL verifiable (criteria 5, 7, 9, 10):**

These need browser-use or human verification for full confidence.
If browser-use is available, take screenshots at key viewports.
If not, score based on code-level analysis and mark as
"(code-level only)".

Criterion 5 — Clickability:
- Check for custom clickable elements (onClick on div/span)
- Verify Button component usage for actions
- VISUAL: hover states, cursor changes

Criterion 7 — Responsive:
- Check for responsive Tailwind classes (sm:, md:, lg:)
- Check Sidebar collapse behavior
- grep for overflow risks:
  - `flex` containers where children lack `min-w-0`
  - Fixed dimensions (`w-[`, `h-[`) inside flex/grid parents
  - Input/Textarea without `w-full` inside containers
  - Missing `overflow-hidden`/`overflow-auto` on scroll containers
  - `whitespace-nowrap` without `overflow-hidden text-ellipsis`
- VISUAL: screenshots at 1440px, 1024px, 768px, 375px
  - Check for horizontal scrollbar on body/main containers
  - Check for elements exceeding parent bounds
  - Check for input fields clipped or extending outside cards/forms

Criterion 9 — Performance:
- Check for Skeleton usage during loading
- Check for unnecessary re-renders or heavy components
- Count clicks for common task flows

Criterion 10 — Accessibility:
- Check for aria-label, aria-live, alt text
- Check heading hierarchy
- Check for color-only indicators
- Search for `text-[a-z-]+/[0-9]+` to find all opacity-modified
  text classes. Classify each as readable-text (violation) or
  decorative (acceptable).
- VISUAL: keyboard navigation test

### Step 3b: Perform visual verification (when browser-use available)

If a development server is running or the application is accessible:

**Required visual checks:**
1. **Responsive breakpoints**: Screenshot key pages at desktop (1440px), tablet (768px), mobile (375px)
2. **Dark mode verification**: Test both light and dark themes if available
3. **Interactive states**: Verify hover, focus, active states on key components
4. **Navigation flows**: Test critical user journeys and form submissions
5. **Accessibility checks**: Keyboard navigation, focus indicators, color contrast

**Dark mode discovery order:**
1. Look for visible theme toggle button/switch in UI
2. Check for `data-theme` attribute or `class="dark"` on html/body elements  
3. Use `prefers-color-scheme` media query override via Chrome DevTools Protocol
4. Document the method used in review metadata

**Partial failure handling:**
- If visual scan fails for specific pages, log the failure and continue with remaining pages
- If dark mode toggle is not discoverable, run light-theme-only screenshots and note "dark mode: not tested" in review metadata
- If dev server is unreachable, continue with code-only review and mark criteria as "(code-level only)"

**Screenshot organization:**
- Save screenshots to `reviews/[timestamp]/screenshots/` directory
- Use naming convention: `[page]-[viewport]-[theme].png`
- Limit to essential screenshots (max 20 per review)

### Step 3c: Merge code and visual findings

Correlate visual findings with code findings where possible:
- If a visual issue has a corresponding code location, report as `[CODE+VISUAL]`
- If visual evidence contradicts code analysis, prioritize visual evidence
- Pure visual findings (no code correlation) should be marked as `[VISUAL]`
- Document any discrepancies between code-level and visual verification

### Step 4: Score each criterion

Before listing findings, apply pattern grouping per the
`<pattern-grouping>` section in the design review skill. Aggregate
findings that share the same root cause into a single [PATTERN]
entry with instance count and consolidated fix prompt.

Apply the quantitative rubric thresholds from the design review skill.
Use the violation counts from Step 3 to determine scores objectively.
Score 1-5 with specific findings per criterion.
For each finding, use the fix-output-format from the design review skill
(file:line, current code, replacement, auto-fixable flag).

### Step 5: Generate the pre-fix report

Use the output format from `.cursor/skills/design-review/SKILL.md`.

If prior reviews exist, add a comparison section:

```
### Score comparison
| Criterion | Previous | Current | Change |
|-----------|----------|---------|--------|
| 1. Aura consistency | 3 | 4 | +1 improved |
| 2. Navigation | 4 | 4 | — unchanged |
| ... | | | |
| **Overall** | **3.2** | **3.8** | **+0.6 improved** |
```

### Step 6: Auto-fix pass

After scoring, apply all auto-fixable issues identified in the scan:

1. Collect all findings marked `auto-fixable: yes` from the report
2. Group by file to minimize file I/O
3. Apply fixes using the editor (StrReplace / file editing)
4. Track each fix: `{ file, line, before, after, criterion }`
5. Log applied fixes in the report under `## Applied fixes`

Skip auto-fix if:
- The finding is marked `auto-fixable: no`
- The fix would change more than 5 lines in a single edit
  (likely needs human review)
- The replacement is not semantically equivalent
  (see `<auto-fix-rules>` in the design review skill)

### Step 7: Re-verify affected criteria

After auto-fixes are applied:

1. Identify which criteria had auto-fixes applied
2. Re-run ONLY those criteria's grep patterns from Step 3
3. Recount violations
4. Re-score using the quantitative rubric thresholds
5. Update the report with a `## Post-fix scores` section:

```
## Post-fix scores
| Criterion | Pre-fix | Post-fix | Change |
|-----------|---------|----------|--------|
| 1. Aura consistency | 3 (47 violations) | 4 (12 violations) | +1 |
| 10. Accessibility | 3 (missing 8 aria) | 4 (missing 2 aria) | +1 |
| **Overall** | **3.7** | **3.9** | **+0.2** |
```

6. Use the post-fix scores as the final scores in history.json

### Step 8: Save the report

1. Create `.design-reviews/` directory if it doesn't exist
2. Write the full report to `.design-reviews/YYYY-MM-DD_review.md`
3. Update `.design-reviews/history.json`:
4. **Screenshot pruning**: After saving, prune screenshot directories older than the last 3 reviews to manage storage

```json
{
  "reviews": [
    {
      "date": "YYYY-MM-DD",
      "overall": 0.0,
      "preFix": 0.0,
      "level": "Excellent|Good|Average|Needs work",
      "scores": {
        "aura-consistency": 0,
        "navigation-hierarchy": 0,
        "labels-language": 0,
        "feedback-validation": 0,
        "clickability": 0,
        "error-prevention": 0,
        "responsive": 0,
        "empty-states": 0,
        "performance": 0,
        "accessibility": 0
      },
      "preFixScores": {
        "aura-consistency": 0,
        "navigation-hierarchy": 0,
        "labels-language": 0,
        "feedback-validation": 0,
        "clickability": 0,
        "error-prevention": 0,
        "responsive": 0,
        "empty-states": 0,
        "performance": 0,
        "accessibility": 0
      },
      "reportFile": "YYYY-MM-DD_review.md",
      "priorityFixes": 0,
      "totalFindings": 0,
      "autoFixesApplied": 0,
      "visualReviewPerformed": false,
      "darkModeTested": "no|yes|partial",
      "viewportsTested": ["desktop", "tablet", "mobile"],
      "visualMethod": "button-toggle|dom-attribute|media-query",
      "screenshotCount": 0,
      "stalledCriteria": ["criterion-name-1", "criterion-name-2"]
    }
  ]
}
```

Note: `overall` and `scores` reflect post-fix values (final).
`preFix` and `preFixScores` preserve the pre-fix state for tracking
improvement from auto-fixes vs. manual effort.

### Step 9: Present results

Show the builder:
1. Post-fix overall score and level
2. Pre-fix vs post-fix comparison (if auto-fixes were applied)
3. Top 3 remaining priority fixes (non-auto-fixable) with prompts
4. Score comparison with prior reviews if history exists
5. List of auto-fixes that were applied
6. Full report location

## Notes

- The review scans the project where the builder is working,
  NOT the Skills repository itself.
- `.design-reviews/` should be added to `.gitignore` or
  committed depending on team preference.
- Each review is cumulative — history.json grows with each run.
- When prior reviews exist in history.json, identify criteria that
  have been stuck at the same score for 2+ reviews. Prioritize
  those criteria's auto-fixes first and flag them as
  "Stalled criteria — needs focused attention" in the report.
- For stalled criteria (5, 7, 9, 10), visual verification often
  reveals issues not detectable through code-only analysis. Consider
  running visual checks specifically for these criteria when they
  remain unchanged across multiple reviews.

## Figma Integration (Future Enhancement)

For design-to-code fidelity verification, a future enhancement will support Figma manifest files:

**Convention**: Place `.figma-manifest.json` in project root:
```json
{
  "designFiles": [
    {
      "fileKey": "abc123def456",
      "name": "Design System Components",
      "pages": [
        {
          "nodeId": "1:23",
          "name": "Login Page",
          "mappedToRoute": "/login"
        }
      ]
    }
  ]
}
```

When present, visual reviews will compare implementation screenshots against Figma designs and report fidelity findings as `[FIGMA-VISUAL]` type. This enables automated design QA at scale.
