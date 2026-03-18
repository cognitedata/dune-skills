# Criterion 8: Empty States — Check Details

## Cross-reference

See the **design-content-guidelines** skill for empty state types and 
patterns:
- Three types: first-use, user-cleared, error/no results
- Pattern: Explanation + CTA to populate
- Tone: hopeful, actionable, guiding
- "Explain why it's empty. Provide clear next action."

## What to check

- Every data-displaying component has an empty state?
  (Table, List, Chart, Count, TreeView, Artifact and sub-variants, Card grids)
  Use Aura Empty State component: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-empty-state--docs
- First-use empty states explain what will appear AND 
  provide a CTA button to create first item?
- Search/filter "no results" states suggest broadening 
  search or clearing filters?
- Empty states match UX Writing tone: hopeful, actionable?
  ("No reports yet. Create your first report to get started." 
  not just "No data")
- Empty states use Aura components (Button for CTA, not 
  just plain text links)?
- First-time user experience is clear without documentation?
