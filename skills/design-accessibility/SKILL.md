---
name: design-accessibility
description: Ensures page-level accessibility for customer-facing apps using Aura — keyboard navigation, headings, alt text, focus management, ARIA attributes. Use when building interactive elements, page layouts, dynamic content, custom components, or managing focus.
allowed-tools: Read, Glob, Grep, Edit, Write
---

## Role
You are building a customer-facing application that must be 
usable by everyone. Aura components handle many accessibility 
concerns automatically, but you are responsible for page-level 
accessibility.

Aura's focus system uses shadow-focus-ring (a custom shadow 
token with inner ring + outer glow). Never remove or override 
this.

For content/writing accessibility (plain language, screen 
reader text, cognitive accessibility), see 
the accessibility guidelines reference in the **design-content-guidelines** skill.


<when-to-reference>
Consult this skill whenever you are:
- Building any interactive element
- Creating or modifying page layouts
- Adding images, icons, or media
- Implementing dynamic content (dialogs, toasts, live updates)
- Building custom components (not Aura)
- Handling focus management
</when-to-reference>

<aura-coverage priority="high">
What Aura components handle automatically:

| Concern | Aura handles | You verify |
|---------|-------------|-----------|
| Focus indicators | shadow-focus-ring on interactive elements | Not hidden by overflow or z-index |
| Keyboard activation | Button: Enter/Space. Input: standard keys | Custom elements also respond |
| ARIA roles | Correct roles on Dialog, Segmented Control (Tabs ARIA pattern), etc. | Custom components have roles |
| Color contrast | Token pairs designed for AA compliance | Page backgrounds don't reduce contrast |
| Dark mode | Semantic tokens adapt automatically | Custom colors also work in dark mode |
| Disabled states | Communicated via aria-disabled | Reason for disabled is accessible |
| Focus trapping | Dialog traps focus when open | You return focus to trigger on close |
</aura-coverage>

<builder-responsibilities priority="critical">

<responsibility name="keyboard-navigation">
<instruction>
- Tab order follows visual reading order
- Every interactive element reachable via Tab
- No keyboard traps
- Skip-to-content link on pages with complex nav
</instruction>

<example type="correct">
<a href="#main-content" className="sr-only focus:not-sr-only 
  focus:absolute focus:top-4 focus:left-4 focus:z-50 
  focus:bg-background focus:px-4 focus:py-2 focus:rounded-md 
  focus:shadow-focus-ring">
  Skip to main content
</a>
<Sidebar />
<main id="main-content">
  <h1>Reports</h1>
  {/* Content in logical tab order */}
</main>
</example>
</responsibility>

<responsibility name="heading-hierarchy">
<instruction>
One H1 per page. Sequential: H1 → H2 → H3. Never skip.
Aura applies text-undefined-foreground to headings by default.
</instruction>
</responsibility>

<responsibility name="image-alt-text">
<instruction>
Every image needs alt. Icons in buttons need aria-label.
</instruction>

<examples>
| Type | Approach | Example |
|------|----------|---------|
| Informational | Describe content | alt="Chart: output up 20%" |
| Decorative | Empty | alt="" |
| Icon button | aria-label on parent | aria-label="Delete report" |
| Icon with label | Hide icon | aria-hidden="true" on icon |
</examples>

<example type="correct">
{/* Icon + text: hide icon from screen reader */}
<Button variant="destructive">
  <TrashIcon className="h-4 w-4 mr-2" aria-hidden="true" />
  Delete report
</Button>

{/* Icon only: label on button */}
<Button variant="ghost" size="icon" aria-label="Delete report">
  <TrashIcon className="h-4 w-4" />
</Button>
</example>
</responsibility>

<responsibility name="dynamic-content">
<examples>
| Scenario | Method |
|----------|--------|
| Search results update | aria-live="polite" |
| Form error | aria-live="assertive" |
| Toast | Sonner handles this |
| Dialog opens | Focus moves to dialog (Aura handles) |
| Dialog closes | Return focus to trigger |
</examples>

<example type="correct">
{/* Screen reader announcement for filtered results */}
<div aria-live="polite" className="sr-only">
  {results.length} results found for "{query}"
</div>
<DataTable data={results} columns={columns} />
</example>
</responsibility>

<responsibility name="color-independence">
<instruction>
Never use color alone to convey meaning.
</instruction>

<example type="correct">
{/* Status with text + color */}
<span className="inline-flex items-center gap-1.5 
  bg-success text-success-foreground text-xs px-2.5 py-0.5 
  rounded-full">
  <CheckCircle className="h-3 w-3" aria-hidden="true" />
  Active
</span>
</example>

<example type="incorrect">
{/* Color only — invisible to colorblind users */}
<span className="h-2 w-2 rounded-full bg-success" />
</example>
</responsibility>

</builder-responsibilities>

<self-check>
Before submitting any page:
- [ ] Tab through all elements in logical order?
- [ ] Every button/link works with Enter/Space?
- [ ] Every dialog opens/closes with keyboard?
- [ ] Escape closes dialogs, popovers, dropdowns?
- [ ] Every image has appropriate alt text?
- [ ] Every form field has a visible label?
- [ ] Non-color indicator for every status?
- [ ] Headings follow H1 → H2 → H3?
- [ ] Dynamic updates announced to screen readers?
- [ ] Focus ring (shadow-focus-ring) visible on all elements?
</self-check>

<edge-cases>
1. Complex data viz? — Text summary via alt or sr-only text.
2. Drag-and-drop? — Keyboard alternative required.
3. Real-time dashboard? — aria-live="polite", not "assertive".
4. Third-party embed? — iframe with descriptive title.
</edge-cases>

<visual-checks>
<instruction>
For accessibility verification, perform these visual tests in both light and dark themes:

**Keyboard navigation verification:**
1. **Tab sequence**: Navigate entire page using only Tab/Shift+Tab keys
2. **Focus indicators**: Every interactive element shows clear focus outline
3. **Skip links**: "Skip to main content" link appears and functions
4. **Keyboard shortcuts**: All interactive elements accessible via keyboard

**Visual contrast and color testing:**
- **Color independence**: All information conveyed by color also available through shape, text, or icons
- **Contrast ratios**: Text maintains 4.5:1 contrast minimum (use browser DevTools or similar)
- **Dark theme**: All text remains readable, focus indicators visible
- **High contrast mode**: UI remains usable in high contrast browser/OS settings

**Screen reader simulation:**
- **Heading structure**: Logical h1→h6 hierarchy without skips
- **Alt text verification**: Images have meaningful descriptions
- **Form labels**: All inputs have associated labels or aria-label
- **Live regions**: Dynamic content changes announced appropriately

**Responsive accessibility:**
- **Touch targets**: Minimum 44px × 44px on mobile devices
- **Zoom testing**: UI remains usable at 200% zoom level
- **Mobile screen reader**: Content accessible with mobile assistive technology
- **Orientation**: UI works in both portrait and landscape modes

**Focus management:**
- **Modal focus trapping**: Focus stays within modal when open
- **Form error focus**: Focus moves to first error field after validation
- **Page navigation**: Focus management on route changes
- **Component state changes**: Focus preserved during dynamic updates
</instruction>

<focus-areas>
Priority visual checks for accessibility:
1. **Keyboard navigation** — Complete tab-through without mouse interaction
2. **Focus visibility** — Clear focus indicators on all interactive elements  
3. **Color contrast** — Verify in both light and dark themes
4. **Screen reader structure** — Logical heading hierarchy and labels
</focus-areas>
</visual-checks>
