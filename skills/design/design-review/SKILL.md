---
name: design-review
description: Evaluates application quality against Aura design standards with scores and Cursor-promptable fix suggestions. Use when running /design-review, evaluating the app, or generating a design review report for human reviewers.
allowed-tools: Read, Glob, Grep, Edit, Write
---

<role>
You are evaluating the quality of a customer-facing 
application against Aura design standards. When scoring, 
be specific — reference exact elements and pages. Every 
finding must include a Cursor-promptable fix suggestion.

This skill is used by the /design-review command and by 
human reviewers in the pre-launch gate.

Related skills (cross-reference when evaluating):
- `../aura-tokens/SKILL.md` — token usage, component mandates, 
  override rules. References Storybook Foundations:
  Colors: https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-colors--docs
  Effects: https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-effects--docs
  Layout: https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-layout--docs
  Typography: https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-typography--docs

- `../aura-components/SKILL.md` — available Aura primitives 
  with Storybook links. Use to verify correct component 
  selection and identify custom components that should be 
  replaced with Aura equivalents.

- `../content-guidelines/SKILL.md` — content quality standards 
  (purposeful, concise, conversational, clear), UX text 
  patterns (buttons, errors, empty states, form fields, 
  notifications), voice and tone framework, accessibility 
  in writing, and research-backed text benchmarks.

- `../layout-patterns/SKILL.md` — approved page layouts.

- `../accessibility/SKILL.md` — page-level a11y 
  requirements, builder responsibilities, and what Aura 
  components handle automatically.

For all Storybook URLs, see `../shared/storybook-links.md`.
</role>

<finding-types>
Classify every finding as one of:
- CODE: Verified by reading source code (file + line)
- GREP: Verified by automated pattern search (count)
- VISUAL: Verified by browser-use screenshot or interaction
- MANUAL: Requires human visual/interaction verification
- PATTERN: Grouped violation appearing in 3+ files 
  (see <pattern-grouping> section)

**Finding correlation model:**
When a VISUAL finding correlates with a CODE finding (e.g., missing hover state detected visually AND missing CSS rule in code), report as a single finding with both evidence types: "[CODE+VISUAL] Component missing hover state - verified in Button.tsx:L45 and browser interaction"

**Auto-fix exclusions:**
VISUAL findings are never auto-fixable (cannot modify code based solely on visual evidence). MANUAL findings require human judgment and are not auto-fixable.

Criteria 1-4, 6, 8 are primarily CODE/GREP verifiable.
Criteria 5, 7, 9, 10 require VISUAL or MANUAL verification 
for full confidence. Mark scores for these criteria with 
"(code-level only)" if no visual checks were performed.
</finding-types>

<when-to-reference>
Consult this skill when:
- Builder runs /design-review command
- You are asked to evaluate the application
- Generating a review report for human reviewers
</when-to-reference>

<scoring-instructions priority="critical">
For each criterion, provide:
1. Score (1-5)
2. Specific observations (exact elements/pages)
3. Fix suggestions as Cursor prompts

Format: [Page] — [Issue] — [Fix prompt]
Example: "Reports page — DataTable has no empty state — 
Add empty state showing 'No reports yet' with Create button"
</scoring-instructions>

<criteria>

<criterion name="aura-consistency" number="1">
<question>Are Aura tokens used correctly throughout?</question>

<cross-reference>
See `../aura-tokens/SKILL.md` for rules.
Storybook Foundations for correct token values.
</cross-reference>

<what-to-check>
- Any hardcoded hex/rgb values in className or style props?
- Any Tailwind default colors (gray-500, red-100, bg-white)?
  These should be Aura semantic tokens (text-muted-foreground, 
  bg-destructive, bg-background). See `../aura-tokens/SKILL.md` 
  common mistakes table.
- All components from Aura library? Check against 
  `../aura-components/SKILL.md` component inventory.
- Any inline style={{}} overrides on Aura components?
- Consistent spacing using Tailwind scale (p-4, gap-6) — 
  no raw pixel values?
- Correct semantic tokens that adapt to dark mode?
  (bg-background not bg-white, text-foreground not text-black)
- Typography using Aura scale? Reference Storybook Typography 
  foundation for correct text-* classes.
- Shadows using Aura effect tokens? Reference Storybook 
  Effects foundation.
- Component consistency: For each interaction pattern 
  (select, dropdown, modal, dialog, popover, tooltip), 
  verify all instances across the app use the same Aura 
  component. Flag any component imported from 
  components/ui/ when an Aura equivalent exists in 
  components/ui/aura/.
</what-to-check>

<rubric>
| Score | Criteria |
|-------|----------|
| 5 | Zero token violations. All semantic tokens. No hardcoded hex/rgb. No Tailwind defaults. No inline style overrides. Dark mode verified. Zero non-Aura component imports where Aura equivalents exist. |
| 4 | <= 15 total token violations. Zero hardcoded hex/rgb. Zero Tailwind default colors. Arbitrary layout values < 20. Inline styles only use CSS custom properties. <= 2 non-Aura component imports with Aura equivalents. |
| 3 | 16-60 total token violations. May have some Tailwind defaults or arbitrary values, but no hardcoded hex colors. 3-5 non-Aura component imports with Aura equivalents. |
| 2 | 61-150 token violations. Hardcoded colors present. Multiple Tailwind default colors. > 5 non-Aura component imports with Aura equivalents. |
| 1 | > 150 token violations. Not using Aura tokens. Custom styling throughout. Aura component library not adopted. |
</rubric>
</criterion>

<criterion name="navigation-hierarchy" number="2">
<question>Can users tell where they are? Clear visual hierarchy?</question>

<cross-reference>
See `../layout-patterns/SKILL.md` for approved page structures.
See `../accessibility/SKILL.md` for heading hierarchy requirements.
See `../aura-components/SKILL.md` for Sidebar, Topbar, 
  Breadcrumb, Segmented Control components.
</cross-reference>

<what-to-check>
- Current page highlighted in Sidebar/Topbar navigation?
- Breadcrumb component used for location context? 
  (https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-breadcrumb--docs)
- Clear heading hierarchy (H1 → H2 → H3, no skips)?
- Page uses an approved layout pattern from `../layout-patterns/SKILL.md`?
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
  from `../layout-patterns/SKILL.md`? (e.g., three-panel: 
  right panel toggleable, left panel collapsible)
- When viewing dense data (charts, timeseries, documents), 
  can the user expand to a fullscreen or wide view?
</what-to-check>

<rubric>
| Score | Criteria |
|-------|----------|
| 5 | Zero heading skips. Breadcrumb on all sub-pages. Active nav item highlighted on every page. One H1 per page. Approved layout pattern used. Content-heavy views maximize primary content area. All secondary panels collapsible/toggleable. Dense data views offer fullscreen/wide mode. |
| 4 | <= 2 heading skips. Breadcrumb on most sub-pages. Active nav item highlighted. Approved layout. Secondary panels collapsible in most multi-panel views. <= 1 content-heavy view missing fullscreen option. |
| 3 | 3-5 heading skips. Breadcrumb missing on some sub-pages. Nav highlight inconsistent. Some secondary panels not collapsible. 2-3 content-heavy views missing real-estate optimization. |
| 2 | > 5 heading skips. No breadcrumb. Nav does not indicate current page. Non-standard layout. Secondary panels fixed/non-collapsible. No fullscreen option for dense data. |
| 1 | No heading structure. No breadcrumb. No navigation highlighting. No approved layout. No layout real-estate optimization. |
</rubric>
</criterion>

<criterion name="labels-language" number="3">
<question>Is all user-facing text clear, purposeful, and well-written?</question>

<cross-reference>
See `../content-guidelines/SKILL.md` for all content standards. 
Specifically:
- Four quality standards: purposeful, concise, conversational, clear
- Button pattern: [Verb] [object] ("Save changes", "Delete report")
- Error pattern: [What failed]. [Why/context]. [What to do.]
- Empty state pattern: Explanation + CTA
- Sentence case for all headings, labels, buttons
- Research benchmarks: buttons 2-4 words, titles 3-6 words,
  8 words = 100% comprehension, 14 words = 90%
</cross-reference>

<what-to-check>
- Buttons follow [Verb] [object] pattern? 
  ("Save changes" not "Submit", "Delete report" not "OK")
- Any generic labels? Check for: Submit, OK, Click here, 
  Yes, No, Confirm, Done, Go, Proceed
- Form fields have visible Label components (not placeholder-only)?
  (https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-label--docs)
- Helper Text used for format hints and constraints?
  (https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-helper-text--docs)
- Language free of jargon and technical terms?
- Consistent terminology across pages? 
  (e.g., same action always uses same verb)
- Sentence case used throughout (not Title Case)?
- Active voice used? (85% active voice target from 
  `../content-guidelines/SKILL.md`)
- Text concise? (Benchmarks: buttons 15-25 chars, 
  titles 30-50 chars, 40-60 chars per line max)
- Page titles specific and descriptive?
  ("Q2 Production reports" not "Reports")
</what-to-check>

<rubric>
| Score | Criteria |
|-------|----------|
| 5 | Zero generic labels. 100% of form fields use Label component. All buttons follow [Verb] [object]. Sentence case throughout. Meets all UX Writing benchmarks. |
| 4 | <= 3 generic labels. >= 90% Label component coverage. <= 2 Title Case headings. Consistent terminology. |
| 3 | 4-10 generic labels. 70-89% Label component coverage. Some placeholder-only fields. Some jargon. |
| 2 | > 10 generic labels. < 70% Label coverage. Heavy technical terms. Inconsistent terminology. |
| 1 | Pervasive generic labels. Missing labels on most fields. Jargon throughout. |
</rubric>
</criterion>

<criterion name="feedback-validation" number="4">
<question>Do users know what's happening? Are forms helpful? Do errors guide recovery?</question>

<cross-reference>
See `../content-guidelines/SKILL.md` for message patterns and 
tone adaptation:
- Error types: validation (inline), system (modal/banner), 
  blocking (full-screen), permission
- Error pattern: [What failed]. [Why/context]. [What to do.]
- Success pattern: [Action] [result/benefit]
- Tone adaptation: frustrated → empathetic, cautious → 
  transparent, confident → efficient
- What to avoid: technical codes, blame language, robotic 
  tone, dead ends, vague causes

See `../aura-components/SKILL.md` for feedback components:
- Alert: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-alert--docs
- Banner: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-banner--docs
- Sonner Toast: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-sonner-toast--docs
- Helper Text: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-helper-text--docs
- Skeleton: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-skeleton--docs
- Progress: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-progress--docs

See `../error-validation/SKILL.md` for implementation patterns.
</cross-reference>

<what-to-check>
- Buttons show loading state during async actions?
  (Check Button component Storybook for loading pattern)
- Pages show Skeleton components while loading?
- Any action >300ms without a loading indicator?
- Forms validate on blur with specific inline messages 
  using Helper Text component?
- Error messages follow [What failed]. [Why]. [What to do.] 
  pattern? (Not "Error", "Invalid input", "Something went wrong")
- No technical error codes exposed to users (403, 500)?
- Success messages use Sonner Toast with specific confirmation?
  ("Report saved successfully" not "Done" or "Success")
- User input preserved on form submission failure?
- Focus moves to first error field on submission failure?
- Appropriate error component used for context?
  (Helper Text for validation, Alert/Banner for system errors, 
  Sonner Toast for brief feedback)
- Every Input/Textarea/Select has a corresponding error state 
  (conditional HelperText with variant="error")?
- Validation strings are field-specific, not generic?
  (See `../error-validation/SKILL.md` message-patterns table)
- Required fields show "required" indicator 
  (asterisk or "(required)" in label)?
- Validation fires on blur AND on submit?
- Form submission is blocked when validation errors exist?
</what-to-check>

<rubric>
| Score | Criteria |
|-------|----------|
| 5 | 100% of async actions have loading states. Zero generic error messages. All forms validate on blur with field-specific messages. Correct feedback component for every context. |
| 4 | >= 90% of async actions have loading states. <= 2 generic error messages. Most forms validate on blur. |
| 3 | 60-89% of async actions have loading states. 3-5 generic errors. Some forms missing validation. |
| 2 | < 60% loading states. > 5 generic errors. Technical error codes exposed (403, 500). Forms lack validation. |
| 1 | No loading states. Silent failures. Technical codes shown to users. No form validation. |
</rubric>
</criterion>

<criterion name="clickability" number="5">
<question>Is it obvious what's clickable? Do interactions feel right?</question>

<cross-reference>
See `../aura-components/SKILL.md` — all interactive elements 
should use Aura components:
- Button: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-button--docs
- Toggle: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-toggle--docs
- Switch: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-switch--docs
- Dropdown Menu: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-dropdown-menu--docs
</cross-reference>

<what-to-check>
- All actions use Aura Button component (not custom styled 
  divs or spans)?
- Correct Button variant chosen? (Check Storybook for variants)
- Hover states present on all interactive elements?
- Cursor changes appropriately (pointer on clickable)?
- No misleading elements (text that looks clickable but isn't,
  or clickable elements that look like static text)?
- Dropdown Menu used for action menus?
- Toggle/Switch used appropriately for state changes?
- InputGroup components have consistent addon sizing?
  (Select/Button addon matches Input height — see 
  `../aura-components/SKILL.md` input-group-sizing)
</what-to-check>

<rubric>
| Score | Criteria |
|-------|----------|
| 5 | Zero onClick on div/span. 100% of actions use Aura Button. Correct variant for every context. Hover states on all interactive elements. |
| 4 | <= 3 onClick-on-div instances. >= 90% Aura Button coverage. Correct variants. Minor hover gaps. |
| 3 | 4-8 onClick-on-div instances. 70-89% Aura Button coverage. Some incorrect variants. |
| 2 | > 8 onClick-on-div instances. < 70% Button coverage. Custom styled clickable elements. |
| 1 | Pervasive custom clickable elements. No Aura Button usage. No hover states. |
</rubric>
</criterion>

<criterion name="error-prevention" number="6">
<question>Warnings before destructive actions? Can users undo or cancel?</question>

<cross-reference>
See `../content-guidelines/SKILL.md` for confirmation dialog content:
- Title: [Action verb] [object]?
- Body: consequences + reversibility
- Confirm button: specific action verb (never Yes/No)
- Tone: serious, transparent, respectful

See `../aura-components/SKILL.md` for confirmation components:
- Alert Dialog: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-alert-dialog--docs
- Dialog: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-dialog--docs

See `../error-validation/SKILL.md` for confirmation patterns.
</cross-reference>

<what-to-check>
- Delete/remove actions show Alert Dialog or Dialog 
  confirmation before executing?
- Confirmation dialog title names the specific action?
  ("Delete this report?" not "Confirm" or "Are you sure?")
- Confirmation uses variant="destructive" on confirm button?
- Confirm button uses specific verb (not "Yes" or "OK")?
- Cancel option clearly available?
- Consequences stated in dialog body?
  ("This cannot be undone" or "You'll lose all data")
- Auto-save implemented where appropriate?
- Forms have Cancel button to discard changes?
</what-to-check>

<rubric>
| Score | Criteria |
|-------|----------|
| 5 | 100% of destructive actions have Alert Dialog with specific verbs. Zero "Yes"/"No"/"OK" confirm buttons. Consequences stated in all dialogs. Cancel on every form. Auto-save where appropriate. |
| 4 | >= 90% of destructive actions confirmed. <= 1 generic confirm button. Cancel available on most forms. |
| 3 | 60-89% confirmed. Some generic confirmations ("Are you sure?"). Some missing cancel options. |
| 2 | < 60% confirmed. "Yes"/"No" dialogs. Missing cancel on most forms. No undo. |
| 1 | No confirmations on destructive actions. Data loss risk. |
</rubric>
</criterion>

<criterion name="responsive" number="7">
<question>Works across screen sizes?</question>

<cross-reference>
See `../layout-patterns/SKILL.md` for responsive behavior per layout.
See `../aura-tokens/SKILL.md` — Storybook Layout foundation for 
  breakpoints and container queries:
  https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-layout--docs
See `../aura-components/SKILL.md`:
- Sidebar: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-sidebar--docs
  (check collapse behavior)
- Drawer: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-drawer--docs
  (mobile navigation alternative)
</cross-reference>

<what-to-check>
- Test at 1440px, 1024px, 768px, 375px widths?
- No horizontal scrolling at any viewport?
- Touch targets 44px+ on mobile?
- Hover-dependent features have touch alternatives?
- Sidebar collapses/hides on mobile?
- Text readable without zooming on mobile?
- Navigation adapts for smaller screens?
- Layout follows responsive behavior defined in 
  `../layout-patterns/SKILL.md` for the chosen pattern?
- Flex children have `min-w-0` to prevent text/input overflow?
- No fixed `w-[Npx]` or `h-[Npx]` inside flex/grid parents?
- Input/Textarea elements use `w-full` inside containers?
- Scroll containers have `overflow-hidden` or `overflow-auto`?
- Elements with `whitespace-nowrap` also have 
  `overflow-hidden text-ellipsis`?
- VISUAL: No elements visually exceeding parent bounds at 
  any tested viewport?
</what-to-check>

<rubric>
| Score | Criteria |
|-------|----------|
| 5 | >= 80% of feature files use responsive breakpoint classes (sm:/md:/lg:). Zero fixed-dimension violations. No horizontal scroll at any viewport. Touch targets 44px+. Layout adapts per pattern specs. |
| 4 | >= 60% responsive class coverage. <= 5 fixed-dimension values. No overflow at tested viewports. Minor touch target issues. |
| 3 | 40-59% responsive class coverage. 6-15 fixed-dimension values. Some layout issues on mobile. |
| 2 | < 40% responsive coverage. > 15 fixed-dimension values. Horizontal scroll on mobile. Layouts break. |
| 1 | No responsive classes. Desktop-only layout. Broken on mobile/tablet. |
</rubric>
</criterion>

<criterion name="empty-states" number="8">
<question>Clear what to do when there's no data?</question>

<cross-reference>
See `../content-guidelines/SKILL.md` for empty state types and 
patterns:
- Three types: first-use, user-cleared, error/no results
- Pattern: Explanation + CTA to populate
- Tone: hopeful, actionable, guiding
- "Explain why it's empty. Provide clear next action."
</cross-reference>

<what-to-check>
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
</what-to-check>

<rubric>
| Score | Criteria |
|-------|----------|
| 5 | 100% of data components have empty states. All three types handled (first-use, cleared, no results). Every empty state has a wired CTA. Follows UX Writing tone. |
| 4 | >= 80% of data components have empty states. <= 2 missing CTAs. Most empty states follow content patterns. |
| 3 | 50-79% of data components have empty states. Some missing no-results states. Some unwired CTAs. |
| 2 | < 50% coverage. Many blank pages. No CTAs on most empty states. |
| 1 | No empty states. Blank pages everywhere. No guidance for users. |
</rubric>
</criterion>

<criterion name="performance" number="9">
<question>Fast? Efficient task completion?</question>

<cross-reference>
See `../aura-components/SKILL.md` for loading components:
- Skeleton: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-skeleton--docs
- Progress: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-progress--docs
- Page Loader: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-page-loader--docs
</cross-reference>

<what-to-check>
- Progressive loading using Skeleton components?
- Full-page loading uses Page Loader component?
- Common tasks achievable in minimal clicks?
- Bulk actions for multi-item operations?
- Defaults pre-filled where appropriate?
- Unnecessary multi-step flows for simple tasks?
- Long operations show Progress component with status?
</what-to-check>

<rubric>
| Score | Criteria |
|-------|----------|
| 5 | Skeleton on all async-loaded surfaces. >= 2 bulk action surfaces. Common tasks <= 3 clicks. Progress component on all operations > 3s. React.lazy on all route-level components. |
| 4 | Skeleton on >= 80% of async surfaces. At least 1 bulk action. Common tasks <= 4 clicks. Most long operations show progress. |
| 3 | Skeleton on 50-79% of async surfaces. No bulk actions. Common tasks require 5+ clicks. Some operations lack progress. |
| 2 | Skeleton on < 50% of surfaces. No bulk actions. Many-step flows for simple tasks. No progressive loading. |
| 1 | No Skeleton usage. No progress indicators. Tedious multi-step flows. |
</rubric>
</criterion>

<criterion name="accessibility" number="10">
<question>WCAG AA 2.1 compliant? Usable with assistive technology?</question>

<cross-reference>
See `../accessibility/SKILL.md` for:
- Builder responsibilities (keyboard nav, headings, alt text, 
  dynamic content, color independence)
- What Aura components handle automatically
- Self-check checklist

See `../content-guidelines/SKILL.md` for content accessibility:
- Screen reader optimization (descriptive labels, link text, 
  ARIA labels)
- Cognitive accessibility (8-14 words per sentence target, 
  scannable chunks, consistent patterns)
- Multi-modal communication (don't rely on color alone, 
  pair indicators with text)
- Plain language (7th-8th grade reading level, avoid idioms)
- Accessible pattern examples for buttons, links, errors, 
  form labels

See also `../content-guidelines/references/accessibility-guidelines.md`
for the comprehensive content accessibility guide.
</cross-reference>

<what-to-check>
- Tab through all elements — logical order per 
  `../accessibility/SKILL.md`?
- All images have appropriate alt text (informational = 
  descriptive, decorative = empty, icon buttons = aria-label)?
- All form fields have visible Label component?
  (https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-label--docs)
- Color never the only indicator? (Pair with text/icon 
  per content-guidelines multi-modal guidelines and 
  `../accessibility/SKILL.md`)
- Focus ring (shadow-focus-ring) visible on all elements?
- Dialog/Alert Dialog traps and returns focus?
  (Check Storybook for each component's a11y behavior)
- Dynamic content changes announced (aria-live)?
- Error messages work with screen readers?
  ("error + field label read together")
- Text meets comprehension benchmarks?
  (8 words = 100%, 14 words = 90%)
- Link text descriptive? 
  ("Read pricing details" not "Click here")
- Interactive elements explicitly labeled?
  ("Submit application" not just "Submit")
- Opacity modifiers on text: Search for Tailwind 
  text-[token]/[opacity] patterns (e.g., 
  text-muted-foreground/50). Any opacity below 100 on 
  text that users need to read is a contrast violation.
  Exceptions: decorative elements (drag handles, 
  background icons) that are not meant to convey 
  information.
</what-to-check>

<rubric>
| Score | Criteria |
|-------|----------|
| 5 | 100% of interactive elements keyboard-accessible. All images have alt text. All form fields have aria-describedby when HelperText present. Zero color-only indicators. All dialogs trap/return focus. aria-live on all dynamic content regions. Zero opacity-modified text classes on readable text. |
| 4 | >= 90% keyboard-accessible. <= 3 missing aria-label/describedby. <= 1 color-only indicator. Most dialogs handle focus correctly. <= 5 opacity-modified text classes (excluding decorative). |
| 3 | 70-89% keyboard-accessible. 4-8 missing ARIA attributes. Some color-only indicators. 6-15 opacity-modified text classes (excluding decorative). Content not optimized for cognitive accessibility. |
| 2 | < 70% keyboard-accessible. > 8 missing ARIA attributes. Multiple color-only indicators. Contrast failures. |
| 1 | No keyboard navigation. No ARIA attributes. No alt text. Inaccessible content. |
</rubric>
</criterion>

</criteria>

<scoring-summary>
Add scores. Divide by 10.

| Average | Level | Action |
|---------|-------|--------|
| 4.5-5.0 | Excellent | Ready to launch |
| 3.8-4.4 | Good | Launch with minor fixes |
| 3.0-3.7 | Average | Fix major issues before launch |
| Below 3.0 | Needs work | Focus on lowest scores first |
</scoring-summary>

<auto-fix-rules>
Critical safety rule: auto-fix ONLY when the replacement is 
semantically equivalent. Never approximate.

Auto-fixable (apply without human review):
- Tailwind arbitrary value -> Tailwind scale ONLY when exact 
  px match exists (e.g. `w-[384px]` -> `w-96` is safe, 
  `w-[400px]` -> `w-96` is NOT — 400 != 384)
- Tailwind default color -> Aura semantic token using the 
  mapping table in `../aura-tokens/SKILL.md` common-mistakes
- Missing aria-label on icon-only buttons
- Missing aria-describedby when HelperText already exists 
  adjacent to the field
- Title Case heading -> sentence case
- Generic button label -> specific [Verb] [object] label 
  (when the action is unambiguous from context)
- Missing role attribute on interactive non-button elements 
  (e.g. onClick div missing role="button")
- text-[token]/[opacity] on readable text elements 
  -> text-[token] (remove opacity modifier). Safe when the 
  element contains user-readable text or meaningful labels. 
  NOT safe for decorative elements (grip handles, background 
  patterns, placeholder icons) — skip those with 
  reason: decorative-element.
- var(--color-X) in style attributes -> var(--X) (remove 
  erroneous "color-" prefix). Always a typo — the Aura 
  token system never uses --color- prefix.

NOT auto-fixable (needs human judgment):
- Arbitrary values with no exact Tailwind scale match 
  (flag for human review with closest alternatives)
- Chart color palette decisions
- Chart series color selection (which --chart-N to use for 
  which series requires understanding the data semantics)
- Replacing var(--primary) or raw ramp var() in chart SVG 
  with --chart-N (requires human judgment on series mapping)
- Responsive layout strategy
- Mobile navigation pattern changes
- Component architecture changes
- New empty state CTA actions
- Any fix that would change more than 5 lines
</auto-fix-rules>

<pattern-grouping>
When the same violation appears in 3+ files, group it as 
a single pattern finding instead of listing each instance.

Format:
- [PATTERN] [description] — [N instances across M files]
  Files: [file1:line, file2:line, ...]
  Fix: [single fix prompt that addresses all instances]

Examples:
- [PATTERN] text-muted-foreground with opacity modifier 
  — 20 instances across 11 files
  Fix: "Search for text-muted-foreground/[0-9]+ across 
  the codebase. For each: if the element is readable text, 
  remove the opacity modifier. If decorative (grip handle, 
  background icon), keep it and add a code comment."

- [PATTERN] Non-Aura component import — 3 instances 
  across 2 files
  Fix: "Replace imports from @/components/ui/popover 
  with @/components/ui/aura/popover in all files."

Pattern findings count as 1 finding in the report 
regardless of instance count. They are scored by the 
total instance count (e.g., 20 opacity violations 
still count as 20 violations in the rubric threshold).
</pattern-grouping>

<fix-output-format>
Each finding must include:
1. Finding type: [GREP], [CODE], [VISUAL], [MANUAL], or 
   [PATTERN] (for grouped violations per <pattern-grouping>)
2. Exact file path and line number(s)
3. Current code snippet (what to find)
4. Replacement code snippet (what to change to)
5. Whether it's auto-fixable (boolean)

Format examples:
- [GREP] [file.tsx:L42] `bg-gray-500` -> `bg-muted` 
  (auto-fixable: yes)
- [CODE] [file.tsx:L100-105] Missing aria-describedby on input 
  -> Add `aria-describedby="helper-id"` to Input and 
  `id="helper-id"` to HelperText (auto-fixable: yes)
- [VISUAL] [LoginPage] Button hover state not visible in dark theme 
  -> Verify hover:bg-primary-hover token implementation 
  (auto-fixable: no - requires visual verification)
- [CODE+VISUAL] [Button.tsx:L23] Missing focus outline detected in tab navigation 
  -> Add focus:ring-2 focus:ring-primary classes 
  (auto-fixable: yes - code fix verified visually)
- [MANUAL] [file.tsx] Mobile layout needs design decision for 
  sidebar collapse behavior (auto-fixable: no)
</fix-output-format>

<output-format>
When generating a review via /design-review:

## Design Quality Review
**Overall: [X.X]/5.0 — [Level]**

### Findings
**1. Aura token consistency — [X]/5** (N violations)
- [GREP] [file:line] `current` -> `replacement` (auto-fixable: yes/no)
- [CODE] [file:line] Description (auto-fixable: yes/no)

**2. Navigation & hierarchy — [X]/5**
- [finding with file:line and auto-fixable flag]

**3. Labels & language — [X]/5**
- [finding with file:line and auto-fixable flag]

**4. Feedback & validation — [X]/5**
- [finding with file:line and auto-fixable flag]

**5. Clickability — [X]/5**
- [finding with file:line and auto-fixable flag]

**6. Error prevention — [X]/5**
- [finding with file:line and auto-fixable flag]

**7. Responsive design — [X]/5**
- [finding with file:line and auto-fixable flag]

**8. Empty states — [X]/5**
- [finding with file:line and auto-fixable flag]

**9. Performance — [X]/5**
- [finding with file:line and auto-fixable flag]

**10. Accessibility — [X]/5**
- [finding with file:line and auto-fixable flag]

### Priority fixes (address before merge)
1. [Most critical finding + file:line + Cursor prompt]
2. [Second + file:line + prompt]
3. [Third + file:line + prompt]

### Applied fixes
- [All auto-fixable findings that were applied, grouped by file]
- [Include criterion, file:line, before -> after for each]

### Skipped auto-fixable
- [Auto-fixable findings that were skipped, with skip reason:
  exceeds-5-lines, no-exact-match, semantic-uncertainty]

### Needs human review
- [All findings marked auto-fixable: no]

### Visual findings without code correlates
- [VISUAL findings that don't have corresponding CODE findings]
- [Include page/component, visual issue description, recommended investigation]

### Review metadata
**Visual review performed:** [yes/no - whether browser-use visual checks were run]
**Dark mode tested:** [yes/no/partial - whether dark theme was successfully verified]
**Viewports tested:** [desktop/tablet/mobile - which responsive breakpoints were checked]
**Visual method:** [button-toggle/dom-attribute/media-query - how dark mode was activated]
**Stalled criteria:** [List of criteria where scores didn't improve between reviews]
**Screenshot count:** [Number of screenshots captured during visual review]
</output-format>
