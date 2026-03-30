# Building pages and layouts

## Role

You are structuring pages for a customer-facing application. Consistent layouts across apps are essential. Every page must use an approved pattern.

The Aura system uses Tailwind CSS for layout. All layouts use Tailwind flex/grid utilities with Aura spacing tokens. The sidebar component uses Aura's sidebar tokens (bg-sidebar, text-sidebar-foreground, etc.).

For all Storybook URLs, see `./storybook-links.md`.

<storybook-foundation>
Source of truth for layout foundations:
https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-layout--docs

Cross-reference these Storybook stories when implementing any layout pattern (full URLs):

| Story | URL | Use for |
|-------|-----|---------|
| Breakpoints | https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--breakpoints | Official breakpoint values |
| Container Queries | https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--container-queries | Responsive within components |
| Column Spans | https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--column-spans | 2-col, 3-col, asymmetric splits |
| Layout Compositions | https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--compositions | Combining layout parts |
| Sidebar Left | https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--sidebar-left-layout | Sidebar implementation |
| Card Grid | https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--card-grid-layout | Card grid layout |
| Dashboard | https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--dashboard-layout | Dashboard with metrics |
| Comprehensive Dashboard | https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--comprehensive-dashboard | Full dashboard |
| Grid Patterns | https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--grid-patterns-reference | Grid configuration catalog |
| Code Examples | https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--code-examples | Copy-paste Tailwind code |

Base: https://cognitedata.github.io/aura/storybook/
</storybook-foundation>

<foundations>
Standard layout primitives used across all patterns:

CONTENT MAX-WIDTHS:
- max-w-7xl — dashboards, full-width layouts
- max-w-4xl — detail pages
- max-w-2xl — forms, wizard step content
- max-w-sm — search inputs, narrow controls

SECTION SPACING:
- space-y-8 — between major page sections (e.g. form groups)
- space-y-6 — between sections within a page
- space-y-4 — between items within a section
- space-y-2 — between label and field, tight groupings

GRID GAPS:
- gap-6 — dashboard grids, chart grids, panel gaps
- gap-4 — card grids, metric grids
- gap-3 — toolbar items, button groups

PAGE PADDING:
- px-6 py-8 — standard content area (desktop)
- px-4 py-6 — mobile content area
- p-4 — card/panel internal padding
- p-6 — larger card internal padding
</foundations>

<sidebar-tokens>
The Aura system has dedicated sidebar tokens that differ from the main content area:

| Token | Purpose | Light value | Dark value |
|-------|---------|-------------|------------|
| bg-sidebar | Sidebar background | mountain-900 | mountain-900 |
| text-sidebar-foreground | Sidebar text | mountain-100 | mountain-100 |
| text-sidebar-primary | Sidebar primary | mountain-600 | mountain-600 |
| text-sidebar-primary-foreground | Active item text | white | white |
| bg-sidebar-accent | Active/hover bg | mountain-700 | mountain-700 |
| text-sidebar-accent-foreground | Active text | white | white |
| border-sidebar-border | Sidebar borders | mountain-800 | mountain-800 |

Note: The sidebar is ALWAYS dark-themed, even in light mode.
</sidebar-tokens>

<patterns>

<pattern name="sidebar-content">
<use-when>
3+ top-level sections. Persistent navigation needed.
Most common for multi-page apps.
</use-when>

<structure>
┌──────────┬─────────────────────────────┐
│          │  Page Header / Breadcrumb   │
│  Sidebar │─────────────────────────────│
│   Nav    │                             │
│  (dark)  │  Main Content Area          │
│          │  (bg-background)            │
│          │                             │
└──────────┴─────────────────────────────┘
</structure>

<responsive-behavior>
Desktop (1440px+): Sidebar 240px, content fills rest.
Tablet (768px-1439px): Sidebar collapsible via hamburger.
Mobile (below 768px): Sidebar hidden. Hamburger menu.
  Consider bottom nav for 3-5 primary sections.
</responsive-behavior>

<storybook-reference>
Implement using Storybook **Example: Sidebar Left**:
https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--sidebar-left-layout
</storybook-reference>
</pattern>

<pattern name="full-width-dashboard">
<use-when>
Data visualizations, metrics, monitoring. Maximum horizontal space needed.
</use-when>

<structure>
┌─────────────────────────────────────────┐
│  Top Navigation Bar                      │
├─────────────────────────────────────────┤
│  Page Header + Filters                   │
├─────────────────────────────────────────┤
│  ┌───────┐  ┌───────┐  ┌───────┐       │
│  │Metric │  │Metric │  │Metric │       │
│  └───────┘  └───────┘  └───────┘       │
├─────────────────────────────────────────┤
│  Charts / Visualizations                 │
├─────────────────────────────────────────┤
│  Data Table                              │
└─────────────────────────────────────────┘
</structure>

<responsive-behavior>
Desktop: Multi-column grid (grid-cols-3 or grid-cols-4).
Tablet: 2-column grid. Charts stack.
Mobile: Single column. Metrics as horizontal scroll.
</responsive-behavior>

<storybook-reference>
Implement using Storybook **Example: Dashboard** and **Example: Comprehensive Dashboard**:
- https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--dashboard-layout
- https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--comprehensive-dashboard
</storybook-reference>
</pattern>

<pattern name="form-page">
<use-when>
Data entry, creation flows, configuration, settings with form fields.
</use-when>

<structure>
┌─────────────────────────────────────────┐
│  Page Header + Back navigation           │
├─────────────────────────────────────────┤
│  ┌───────────────────────────────┐      │
│  │  Form Section 1 (heading)     │      │
│  │  [fields]                     │      │
│  ├───────────────────────────────┤      │
│  │  Form Section 2 (heading)     │      │
│  │  [fields]                     │      │
│  └───────────────────────────────┘      │
├─────────────────────────────────────────┤
│  Sticky footer: [Cancel]  [Save action] │
└─────────────────────────────────────────┘
</structure>

<responsive-behavior>
Desktop: Form centered, max-w-2xl (672px) or max-w-3xl.
Tablet: Form fills width with px-6 padding.
Mobile: Full width. Sticky footer stays. Fields stack.
</responsive-behavior>

<storybook-reference>
Use centered content (max-w-2xl) and Tailwind patterns from Storybook **Code Examples**:
https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--code-examples
</storybook-reference>
</pattern>

<pattern name="detail-page">
<use-when>
Viewing a single record: report details, user profile, item information with related data.
</use-when>

<structure>
┌─────────────────────────────────────────┐
│  Breadcrumb: Reports > Q2 Summary        │
├─────────────────────────────────────────┤
│  Record Header [Title, status, actions]  │
├─────────────────────────────────────────┤
│  ┌─────────────────┬───────────────┐    │
│  │  Main Content    │  Sidebar      │    │
│  │  (2/3 width)     │  (1/3 width)  │    │
│  └─────────────────┴───────────────┘    │
└─────────────────────────────────────────┘
</structure>

<responsive-behavior>
Desktop: Two-column (grid-cols-3, main span-2, sidebar span-1).
Tablet: Sidebar below main content.
Mobile: Single column. Sidebar collapses.
</responsive-behavior>

<storybook-reference>
Use asymmetric columns from Storybook **Pattern: Column Spans** and composition patterns from **Code Examples**:
- https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--column-spans
- https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--code-examples
</storybook-reference>
</pattern>

<pattern name="settings-page">
<use-when>
App preferences, account settings, notification config.
</use-when>

<structure>
┌─────────────────────────────────────────┐
│  Page Header: Settings                   │
├───────────┬─────────────────────────────┤
│  Settings │  Section Content            │
│   Nav     │  [Form fields / toggles]    │
└───────────┴─────────────────────────────┘
</structure>

<responsive-behavior>
Desktop: Left nav + content area.
Tablet: Top tabs replacing left nav.
Mobile: Category list → tap opens section full-screen.
</responsive-behavior>

<storybook-reference>
Implement using Storybook **Pattern: Layout Compositions**:
https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--compositions
</storybook-reference>
</pattern>

<pattern name="split-screen">
<use-when>
Comparison views, editor + preview, master-detail with equal emphasis on both sides.
</use-when>

<structure>
┌─────────────────────┬─────────────────────┐
│                     │                     │
│   Panel Left        │   Panel Right       │
│   (1/2 width)       │   (1/2 width)       │
│                     │                     │
└─────────────────────┴─────────────────────┘
</structure>

<responsive-behavior>
Desktop: grid-cols-2, equal columns.
Tablet: grid-cols-2 with narrower gap.
Mobile: Stack vertically (grid-cols-1), or use Segmented Control to switch between panels.
</responsive-behavior>

<storybook-reference>
Implement using Storybook **Pattern: Column Spans**:
https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--column-spans
</storybook-reference>
</pattern>

<pattern name="three-panel">
<use-when>
Navigation + content + properties panel. IDE-style layouts. Complex editing workflows with context panels.
</use-when>

<structure>
┌──────────┬───────────────────┬──────────┐
│          │                   │          │
│  Nav/    │   Main Content    │  Props/  │
│  Tree    │   (flexible)      │  Detail  │
│  (fixed) │                   │  (fixed) │
│          │                   │          │
└──────────┴───────────────────┴──────────┘
</structure>

<responsive-behavior>
Desktop (1440px+): All 3 panels visible.
Tablet (768-1439px): Hide right panel, toggle via button.
Mobile (below 768px): Single panel with navigation as Drawer, right panel as bottom sheet or separate route.
</responsive-behavior>

<storybook-reference>
Implement using Storybook **Pattern: Layout Compositions**:
https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--compositions
</storybook-reference>
</pattern>

<pattern name="list-page">
<use-when>
Browsing collections — reports, users, assets, items. The most common page type in data-heavy applications.
</use-when>

<structure>
┌──────────────────────────────────────────┐
│  Page Header [Title]    [Create button]  │
├──────────────────────────────────────────┤
│  Filters toolbar  [Search] [Filters]     │
├──────────────────────────────────────────┤
│  Table / List                            │
│  (with empty state when no data)         │
├──────────────────────────────────────────┤
│  Pagination                              │
└──────────────────────────────────────────┘
</structure>

<responsive-behavior>
Desktop: Full table with all columns visible.
Tablet: Hide non-essential columns, allow horizontal scroll.
Mobile: Switch to card/list view with stackable filters.
</responsive-behavior>

<storybook-reference>
Use Storybook **Example: Card Grid** for card-style list variants and **Grid Patterns** for table/grid configuration:
- https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--card-grid-layout
- https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--grid-patterns-reference
</storybook-reference>
</pattern>

<pattern name="wizard">
<use-when>
Multi-step creation flows, onboarding, configuration wizards, setup processes.
</use-when>

<structure>
┌──────────────────────────────────────────┐
│  Step indicator (1 — 2 — 3 — 4)         │
├──────────────────────────────────────────┤
│                                          │
│   Step Content Area                      │
│   (centered, max-w-2xl)                  │
│                                          │
├──────────────────────────────────────────┤
│  [Back]                    [Next/Submit] │
└──────────────────────────────────────────┘
</structure>

<responsive-behavior>
Desktop: Centered content, horizontal numbered step indicator.
Tablet: Same layout with px-6 padding.
Mobile: Step indicator becomes compact ("Step 2 of 4"), content fills width.
</responsive-behavior>

<storybook-reference>
Center step content with max-w-2xl; use Storybook **Code Examples** for step and footer patterns:
https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--code-examples
</storybook-reference>
</pattern>

</patterns>

<edge-cases>
1. Only 1-2 pages? — Sidebar still works, or use top nav.
2. Layout not listed? — Compose from these patterns. Add a short code comment if the composition is genuinely new.
3. Both data display and entry? — Choose by primary purpose.
4. Need resizable panels? — Start with split-screen or three-panel, add resize handles as needed.
5. Very wide content (data tables)? — Use full-width-dashboard without max-w constraint, or list-page with horizontal scroll.
</edge-cases>
