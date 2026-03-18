# Criterion 2: Navigation & Hierarchy — Check Details

## Cross-reference

See the **design-layout-patterns** skill for approved page structures.
See the **design-accessibility** skill for heading hierarchy requirements.
See the **design-aura-components** skill for Sidebar, Topbar, 
Breadcrumb, Segmented Control components.

## What to check

- Current page highlighted in Sidebar/Topbar navigation?
- Breadcrumb component used for location context? 
  (https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-breadcrumb--docs)
- Clear heading hierarchy (H1 → H2 → H3, no skips)?
- Page uses an approved layout pattern from **design-layout-patterns** skill?
- F/Z-pattern reading flow?
- Related items visually grouped with consistent spacing?
- Segmented Control used correctly for within-page sections?
  (https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-segmented-control--docs)
- For content-heavy views (detail pages, investigation 
  workspaces, document viewers), is the primary content 
  area given maximum available space? Are secondary panels 
  (sidebars, assistant panels, property panels) 
  collapsible or toggleable?
- Does multi-panel layout follow the responsive behavior 
  from **design-layout-patterns** skill? (e.g., three-panel: 
  right panel toggleable, left panel collapsible)
- When viewing dense data (charts, timeseries, documents), 
  can the user expand to a fullscreen or wide view?
