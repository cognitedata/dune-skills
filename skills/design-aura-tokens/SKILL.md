---
name: design-aura-tokens
description: Enforces Aura design tokens via Tailwind — semantic colors, typography, spacing, shadows, no hardcoded values. Use when writing or modifying styles, choosing colors/spacing/typography, reviewing code for design compliance, or responding to style adjustment requests.
allowed-tools: Read, Glob, Grep, Edit, Write
---

## Role
You are building a customer-facing industrial application using 
the Aura design system. Aura is built on Tailwind CSS with 
custom design tokens defined as CSS custom properties and exposed 
as Tailwind utility classes.

This skill contains the non-negotiable design system rules. 
For the full token reference (values, scales, examples), always 
consult the Aura Storybook foundations pages linked below.

For all Storybook URLs, see `./storybook-links.md`.


<when-to-reference>
Consult this skill whenever you are:
- Writing or modifying CSS, styles, or visual properties
- Choosing colors, spacing, typography, or shadows
- Choosing or creating UI components
- Responding to requests like "make it look better"
- Reviewing existing code for design compliance
</when-to-reference>

<storybook-foundations>
For token values, scales, and visual examples, reference:

Colors: https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-colors--docs
Effects: https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-effects--docs
Layout: https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-layout--docs
Typography: https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-typography--docs

These are the source of truth for all token values. Do not 
hardcode values — look up the correct token in Storybook.
</storybook-foundations>

<critical-concept>
TAILWIND CLASSES, NOT RAW CSS VARIABLES

Aura tokens are consumed via Tailwind utility classes. 
Never write raw CSS with var(--token-name) in component code.

WRONG: style={{ color: 'var(--foreground)' }}
WRONG: style={{ color: '#1a1a1a' }}
RIGHT: className="text-foreground"

WRONG: style={{ backgroundColor: 'var(--destructive)' }}
RIGHT: className="bg-destructive"

The only place raw CSS variables appear is in global.css
and in SVG/chart attributes where Tailwind classes cannot 
apply (see chart-and-svg-exceptions rule below).
Application code always uses Tailwind classes.
</critical-concept>

<rules priority="critical">

<rule name="semantic-tokens-only">
<instruction>
All colors must use Aura's SEMANTIC tokens, not raw color 
ramp values or Tailwind defaults. Semantic tokens adapt to 
light/dark mode automatically.

Reference the Colors foundation in Storybook for the full 
semantic token list and their light/dark mode values.
</instruction>

<rationale>
Semantic tokens like bg-background and text-foreground resolve 
to different colors in light vs dark mode. Raw ramp values 
(bg-mountain-950) or Tailwind defaults (bg-gray-500) do not 
adapt and will break the dark mode experience.
</rationale>

<key-semantic-tokens>
These are the most commonly used. For the complete list, 
see Storybook Colors foundation.

BACKGROUNDS: bg-background, bg-card, bg-popover, bg-muted, 
  bg-accent, bg-hover
ACTIONS: bg-primary, bg-secondary, bg-tertiary, bg-destructive
  (each has a -hover variant)
STATUS: bg-destructive, bg-warning, bg-success, bg-neutral, 
  bg-undefined, bg-disabled
TEXT: text-foreground, text-muted-foreground, text-card-foreground,
  text-primary-foreground, text-destructive-foreground,
  text-warning-foreground, text-success-foreground,
  text-foreground-link
BORDERS: border-border, border-input
FOCUS: shadow-focus-ring, shadow-focus-ring-destructive
DECORATIVE: bg-decorative-background-[color], 
  text-decorative-foreground-[color]
  (mountain, fjord, nordic, aurora, dusk, orange, sky, gray)
CHART: chart-[series]-solid, chart-[series]-opacity-[1-5]
  (fjord, aurora, nordic, dusk, orange)
</key-semantic-tokens>

<common-mistakes>
| What people write | What they should write | Why |
|------------------|----------------------|-----|
| text-gray-500 | text-muted-foreground | Tailwind default, not Aura |
| bg-red-100 | bg-destructive | Tailwind default, not Aura |
| bg-green-100 | bg-success | Tailwind default, not Aura |
| text-black | text-foreground | Hardcoded, no dark mode |
| bg-white | bg-background | Hardcoded, no dark mode |
| border-gray-200 | border-border | Tailwind default, not Aura |
| text-blue-500 | text-foreground-link | Tailwind default, not Aura |
| style={{ color: '#333' }} | className="text-foreground" | Inline hardcoded |
</common-mistakes>

<example type="correct">
<div className="bg-card border border-border rounded-lg 
               shadow-sm p-6 space-y-4">
  <h3 className="text-lg font-semibold text-foreground">
    Q2 Production summary
  </h3>
  <span className="bg-success text-success-foreground 
                   text-xs px-2.5 py-0.5 rounded-full">
    Active
  </span>
  <p className="text-sm text-muted-foreground">
    Last updated 2 hours ago
  </p>
</div>
</example>

<example type="incorrect">
<div style={{ 
  backgroundColor: '#f9fafa', border: '1px solid #e4e6e8',
  borderRadius: '8px', padding: '24px'
}}>
  <h3 style={{ color: '#111213' }}>Q2 Production summary</h3>
  <span className="bg-green-100 text-green-500">Active</span>
</div>
</example>
</rule>

<rule name="no-hardcoded-values">
<instruction>
Never use hardcoded hex colors, RGB values, raw CSS color 
values, or Tailwind's default color palette anywhere in 
component code.

For spacing, typography, effects, and radius values, 
reference the corresponding Storybook foundation pages.
</instruction>
</rule>

<rule name="typography-from-storybook">
<instruction>
Use Aura's typography scale exclusively. The system overrides 
several Tailwind defaults with tighter line heights.

Reference: https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-typography--docs

Key font families:
- font-sans (Inter) — all application UI text
- font-marketing (Space Grotesk) — marketing pages only
- font-mono (Source Code Pro) — code, technical values
</instruction>
</rule>

<rule name="effects-from-storybook">
<instruction>
Use Aura's shadow and effect tokens for all elevation.
Reference: https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-effects--docs

Never write custom box-shadow values.
</instruction>
</rule>

<rule name="layout-from-storybook">
<instruction>
Use Aura's layout foundations for spacing and sizing.
Reference: https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-layout--docs

Use Tailwind spacing scale (p-4, gap-6, etc.) — 
never raw pixel values.
</instruction>
</rule>

<rule name="aura-components-only">
<instruction>
For any UI element that has an Aura component equivalent, 
use the Aura component. Do not build custom replacements.

See **design-aura-components** skill for the full list of 
available components with Storybook links.
</instruction>

<rationale>
Aura components have built-in accessibility, correct token 
usage, dark mode support, and consistent styling. Custom 
components lose all of this.
</rationale>
</rule>

<rule name="input-group-sizing">
<instruction>
InputGroup addons (prefix/suffix Select, Button, text) must 
use the same size variant as the Input they are attached to. 
Never mix size variants within an InputGroup.

The dropdown/Select trigger width should fit its content — 
do not stretch it to match the input width.
</instruction>

<example type="incorrect">
{/* Mismatched sizes — Select is default, Input is sm */}
<InputGroup>
  <Select size="default">...</Select>
  <Input size="sm" />
</InputGroup>
</example>

<example type="correct">
{/* Consistent sizing */}
<InputGroup>
  <Select size="sm">...</Select>
  <Input size="sm" />
</InputGroup>
</example>
</rule>

<rule name="no-style-overrides">
<instruction>
Never apply inline styles or override Aura component 
appearances. No style={{}} for visual properties, no custom 
className overrides, no !important, no CSS targeting internals.

The only acceptable inline styles are for truly dynamic 
values (calculated widths, positioned overlays).
</instruction>
</rule>

<rule name="chart-and-svg-exceptions">
<instruction>
SVG elements and charting libraries (recharts, d3, 
Highcharts, etc.) cannot use Tailwind utility classes for 
stroke, fill, and similar attributes. In these contexts, 
using var() references to CSS custom properties is the 
correct approach.

Allowed patterns in SVG/chart style attributes:
- var(--chart-1) through var(--chart-5) for data series
- var(--muted-foreground), var(--border), var(--foreground) 
  for axes, grid lines, labels
- var(--background) for chart backgrounds, knockout areas

NOT allowed in SVG/chart style attributes:
- Raw ramp variables: var(--orange-500), var(--fjord-600)
  -> Use var(--chart-N) instead
- var(--primary) for chart series
  -> --primary is for buttons/actions, not data colors
- var(--color-X) prefix -> typo, should be var(--X)
- Hardcoded hex values (#3b82f6)
  -> Use the appropriate var(--chart-N) token
</instruction>

<common-mistakes>
| What people write | What they should write | Why |
|---|---|---|
| stroke="var(--orange-500)" | stroke="var(--chart-4)" | Raw ramp, not semantic |
| fill="var(--primary)" | fill="var(--chart-1)" | --primary is for UI actions |
| stroke="var(--color-muted-foreground)" | stroke="var(--muted-foreground)" | --color- prefix doesn't exist |
| fill="#3b82f6" | fill="var(--chart-1)" | Hardcoded hex |
</common-mistakes>
</rule>

</rules>

<rules priority="high">

<rule name="dark-mode-awareness">
<instruction>
Aura supports light and dark mode. Semantic tokens adapt 
automatically. Key awareness:
1. Never assume white background — use bg-background
2. Never assume dark text — use text-foreground
3. The sidebar is ALWAYS dark (mountain-900) in both modes
4. For conditional dark styling: dark:bg-mountain-800

Reference Colors foundation for light/dark value pairs.
</instruction>
</rule>

<rule name="heading-hierarchy">
<instruction>
Use heading levels (H1-H6) in strict sequential order.
- One H1 per page (the page title)
- Never skip levels (H1 directly to H3)
- Never use heading tags for visual sizing — use text-* 
  classes from the Typography foundation instead
</instruction>
</rule>

<rule name="empty-states-required">
<instruction>
Every component that displays data must handle the empty state.
Each empty state must include:
1. A message explaining what will appear here
2. A call to action (button or guidance)
Use the Aura Empty State component (**design-aura-components** skill).
See Storybook: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-empty-state--docs
See **design-content-guidelines** skill for copy patterns.
</instruction>
</rule>

<rule name="approved-layouts-only">
<instruction>
Every page must use an approved layout pattern from 
**design-layout-patterns** skill. Do not invent new page layouts.
</instruction>
</rule>

</rules>

<edge-cases>
1. Can it be done with semantic tokens?
   — Always prefer bg-background, text-foreground, etc.
   — Check Storybook Colors foundation for the right token.

2. Need a color with no semantic token?
   — Check decorative tokens first.
   — For charts, use chart-* tokens.
   — Raw ramp values only as last resort, with REVIEW comment.

3. Need Tailwind's default colors (gray-500, blue-600)?
   — Almost always wrong. Map to Aura equivalent.
   — Check Storybook Colors for the correct semantic token.

4. Unsure which token to use?
   — Look it up in Storybook. The foundations pages show 
     every token with its purpose and visual example.

Flag anything unresolvable:
{/* REVIEW: No Aura token for [need]. 
    Using [temporary approach]. */}
</edge-cases>
