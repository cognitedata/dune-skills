---
name: aura-components
description: Selects and implements Aura design system components; always prefer Aura over custom UI. Use when creating interactive UI, choosing components, building forms/tables/navigation/data display, or checking component accessibility.
allowed-tools: Read, Glob, Grep, Edit, Write
---

<role>
You are selecting and implementing UI components for a 
customer-facing application. The Aura design system provides 
a complete component library. Your job is to use the right 
component for each need — never build custom when Aura covers it.

For props, variants, code examples, and interactive demos, 
always reference the Storybook documentation linked below.
This skill handles the decision-making and builder 
responsibilities that Storybook does not cover.

For all Storybook URLs, see `../shared/storybook-links.md`.
</role>

<when-to-reference>
Consult this skill whenever you are:
- Creating any interactive UI element
- Deciding which component to use for a specific need
- Unsure whether Aura has a component for something
- Checking what accessibility a component handles vs. what 
  you need to add yourself
</when-to-reference>

<usage-principles priority="critical">
1. Always check this inventory before building anything custom.
2. Use the component as documented in Storybook — do not 
   override its styles (see `../aura-tokens/SKILL.md`).
3. Choose the correct variant for context (e.g., destructive 
   button for delete actions).
4. Provide descriptive labels following `../content-guidelines/SKILL.md`.
5. For full props API and interactive demos, click the 
   Storybook link for each component.
</usage-principles>

<available-components>

ACTIONS AND INPUTS
| Component | Use for | Storybook |
|-----------|---------|-----------|
| Button | Triggering actions (save, delete, create) | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-button--docs |
| Button Group | Grouping related actions | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-button-group--docs |
| Input | Single-line text entry | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-input--docs |
| Input Group | Composed input with addons (see sizing rule below) | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-input-group--docs |
| Textarea | Multi-line text entry | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-textarea--docs |
| Select | Dropdown selection (4+ options) | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-select--docs |
| Combobox | Searchable selection (long lists) | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-combobox--docs |
| Checkbox | Multi-select toggles | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-checkbox--docs |
| Radio Group | Single selection (2-3 options) | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-radio-group--docs |
| Switch | On/off toggle | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-switch--docs |
| Toggle | Toggle button state | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-toggle--docs |
| Slider | Range / numeric selection | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-slider--docs |
| Date Picker | Date selection | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-date-picker--docs |
| Time Input | Time entry | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-time-input--docs |
| Calendar | Calendar display/selection | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-calendar--docs |

FORM SUPPORT
| Component | Use for | Storybook |
|-----------|---------|-----------|
| Label | Form field labels | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-label--docs |
| Helper Text | Hint/error text below fields | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-helper-text--docs |

LAYOUT AND CONTAINERS
| Component | Use for | Storybook |
|-----------|---------|-----------|
| Card | Content containers | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-card--docs |
| Accordion | Expandable content sections | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-accordion--docs |
| Collapsible | Single collapsible section | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-collapsible--docs |
| Separator | Visual divider | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-separator--docs |
| Swap Slot | Animated content swap | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-swap-slot--docs |
| Empty State | No-data placeholder with CTA | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-empty-state--docs |

NAVIGATION
| Component | Use for | Storybook |
|-----------|---------|-----------|
| Sidebar | App-level navigation | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-sidebar--docs |
| Topbar | Top navigation bar | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-topbar--docs |
| Breadcrumb | Location context | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-breadcrumb--docs |
| Segmented Control | Segmented content switching | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-segmented-control--docs |
| Menubar | Menu bar navigation | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-menubar--docs |
| Pagination | Paged data navigation | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-pagination--docs |
| Toolbar | Action bar for tools | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-toolbar--docs |

DATA DISPLAY
| Component | Use for | Storybook |
|-----------|---------|-----------|
| Table | Tabular data | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-table--docs |
| List | List data display | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-list--docs |
| Badge | Status indicators, labels | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-badge--docs |
| Avatar | User/entity representation | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-avatar--docs |
| Progress | Progress indicators | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-progress--docs |
| Skeleton | Loading placeholders | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-skeleton--docs |
| Kbd | Keyboard shortcut display | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-kbd--docs |
| TreeView | Hierarchical data with status indicators | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-treeview--docs |

ARTIFACT COMPONENTS (domain-specific)
All Artifact sub-variants share the same Storybook docs page. Chart and Count also exist as
standalone components for use outside an Artifact container.

| Component | Use for | Storybook |
|-----------|---------|-----------|
| Artifact (Count) | Metric/count display inside an Artifact | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-artifact--docs |
| Artifact (List) | List display inside an Artifact | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-artifact--docs |
| Artifact (Progress) | Progress tracking inside an Artifact | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-artifact--docs |
| Artifact (Alert) | Alert/status inside an Artifact | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-artifact--docs |
| Artifact (Tree View) | Hierarchical data inside an Artifact | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-artifact--docs |
| Artifact (Chart) | Data visualization inside an Artifact | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-artifact--docs |
| Chart | Standalone data visualization (pie, stacked area, multi-line, horizontal bar, radial) | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-chart--docs |
| Count | Standalone metric/count display | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-count--docs |

FEEDBACK AND OVERLAYS
| Component | Use for | Storybook |
|-----------|---------|-----------|
| Alert | Inline system messages | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-alert--docs |
| Alert Dialog | Confirmation requiring action | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-alert-dialog--docs |
| Banner | Page-level notifications | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-banner--docs |
| Dialog | Modal dialogs | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-dialog--docs |
| Drawer | Slide-out panel | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-drawer--docs |
| Sonner Toast | Brief auto-dismiss feedback | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-sonner-toast--docs |
| Tooltip | Hover/focus contextual help | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-tooltip--docs |
| Popover | Click-triggered rich content | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-popover--docs |
| Hover Card | Hover-triggered rich preview | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-hover-card--docs |
| Context Menu | Right-click menu | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-context-menu--docs |
| Dropdown Menu | Action menu | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-dropdown-menu--docs |
| Command | Command palette / search | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-command--docs |
| Page Loader | Full-page loading animation | https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-page-loader--docs |

> For AI-powered features (chatbots, copilots, agent UIs), see the AI Components section in `../shared/storybook-links.md`.

</available-components>

<decision-tree>
Use this when unsure which component to use:

Need to trigger an action?
  → Button (see Storybook for variants: default, secondary, 
    tertiary, destructive, outline, ghost, link)
  → Button Group for related actions

Need text input?
  Single line → Input
  Multi-line → Textarea
  With addons (icons, prefixes) → Input Group

Need selection?
  One from 2-3 options → Radio Group
  One from 4+ options → Select
  One from long searchable list → Combobox
  Multiple from a list → Checkbox group
  On/off → Switch
  Date → Date Picker
  Time → Time Input
  Range → Slider

Need to display data?
  Tabular → Table
  List format → List (standalone) or Artifact (List) inside an Artifact container
  Metric/count → Count (standalone) or Artifact (Count) inside an Artifact container
  Chart → Chart (standalone) or Artifact (Chart) inside an Artifact container
  Progress → Progress (standalone) or Artifact (Progress) inside an Artifact container
  Alert within an Artifact → Artifact (Alert)
  Hierarchical data → TreeView (standalone) or Artifact (Tree View) inside an Artifact container
  Key-value pairs → Card with description list

Need to show a message?
  Inline, persistent → Alert
  Page-level banner → Banner
  Brief, auto-dismiss → Sonner Toast
  Requires user action → Alert Dialog
  Complex content/form → Dialog

Need contextual help?
  On hover/focus, short → Tooltip
  On hover, rich preview → Hover Card
  On click, rich content → Popover
  Below a form field → Helper Text

Need a menu?
  Primary actions for item → Dropdown Menu
  Right-click context → Context Menu
  Search + actions → Command

Need navigation?
  App-level sections → Sidebar
  Top bar → Topbar
  Location context → Breadcrumb
  Within-page sections → Segmented Control
  Paged data → Pagination

Need loading state?
  Full page/section → Skeleton
  Full-page loading animation → Page Loader
  Inline progress → Progress
  Long operation → Progress with status text

Need an empty state?
  No data component → Empty State

Need layout structure?
  Content container → Card
  Expandable sections → Accordion or Collapsible
  Visual divider → Separator
  Slide-out panel → Drawer
</decision-tree>

<input-group-sizing priority="high">
When composing InputGroup with addons (Select, Button, text):
- All addons must share the same height as the Input. Never 
  mix size variants within a single InputGroup.
- Dropdown/Select trigger width should match its content, 
  not stretch to the full input width.
- Verify consistent sizing visually — a mismatched addon 
  height is a common builder mistake.
See Storybook for all InputGroup variants:
https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-input-group--docs
</input-group-sizing>

<builder-responsibilities priority="critical">
These apply to ALL components. For component-specific 
accessibility details, check each component's Storybook page.

1. LABELS: All interactive elements need descriptive labels.
   Buttons: [Verb] + [Noun] pattern (see `../content-guidelines/SKILL.md`).
   Icon-only buttons: add aria-label.
   Form fields: always use Label component.

2. VARIANTS: Choose the correct variant for context.
   Destructive actions → variant="destructive"
   Primary page action → variant="default" (one per section)
   Supporting actions → variant="secondary"

3. EMPTY STATES: Every data-displaying component (Table, List, 
   Chart, Count, Artifact and its sub-variants) must handle empty 
   state. Use the Aura Empty State component where possible.

4. LOADING STATES: Show Skeleton or Button loading state 
   for any async operation.

5. CONFIRMATION: Destructive actions must show Alert Dialog 
   or Dialog before executing.
</builder-responsibilities>

<edge-cases>
1. Need something not in the list?
   — Check if it can be composed from existing components.
   — If genuinely new, build with Aura tokens and flag:
   {/* REVIEW: No Aura component for [need]. Custom build. */}

2. Component almost fits but not quite?
   — Check Storybook for all variants and props first.
   — Do NOT override styles. Flag for design system team.

3. Choosing between similar components?
   — Dialog vs Alert Dialog: Alert Dialog for simple 
     confirm/cancel. Dialog for complex content.
   — Select vs Combobox: Combobox when list is long or 
     needs search. Select for short fixed lists.
   — Tooltip vs Popover: Tooltip for one line of text.
     Popover for rich content that needs interaction.
</edge-cases>

<visual-checks>
<instruction>
For Aura component verification, perform these interactive visual tests:

**Component state verification:**
1. **Button states**: Test hover, active, focus, disabled states for all button variants
2. **Form components**: Verify default, focus, error, disabled states on inputs, selects, textareas
3. **Interactive feedback**: Hover effects on clickable elements (cards, table rows, menu items)
4. **Loading states**: Spinners, skeleton screens, progress indicators display correctly

**Component variant consistency:**
- **Button variants**: Primary, secondary, outline, ghost render with consistent styling
- **Size variants**: sm, md, lg variants maintain proportional spacing and typography
- **Color variants**: destructive, success, warning variants use correct design tokens
- **Icon consistency**: Icons aligned properly within buttons, consistent sizing

**Component behavior verification:**
- **Dropdown/Select**: Click to open, keyboard navigation, selection behavior
- **Modal/Dialog**: Open/close animations, backdrop behavior, focus trapping
- **Tooltip/Popover**: Trigger on hover/click, positioning, dismiss behavior
- **Navigation**: Tab highlighting, breadcrumb links, pagination controls

**Cross-component consistency:**
- Same interaction patterns (e.g., all selects) behave identically
- Consistent spacing, typography, and color usage across similar components
- Icons and visual indicators follow the same conventions
- Error states display consistently across different form components
</instruction>

<focus-areas>
Priority visual checks for Aura components:
1. **Interactive states** — Hover, focus, active states must be clearly differentiated
2. **Variant consistency** — All size/color variants should feel cohesive
3. **Component behavior** — Dropdowns, modals, tooltips function as expected
4. **Cross-component patterns** — Similar components should behave similarly
</focus-areas>
</visual-checks>
