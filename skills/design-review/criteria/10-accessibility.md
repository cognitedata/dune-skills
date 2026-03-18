# Criterion 10: Accessibility — Check Details

## Cross-reference

See the **design-accessibility** skill for:
- Builder responsibilities (keyboard nav, headings, alt text, 
  dynamic content, color independence)
- What Aura components handle automatically
- Self-check checklist

See the **design-content-guidelines** skill for content accessibility:
- Screen reader optimization (descriptive labels, link text, 
  ARIA labels)
- Cognitive accessibility (8-14 words per sentence target, 
  scannable chunks, consistent patterns)
- Multi-modal communication (don't rely on color alone, 
  pair indicators with text)
- Plain language (7th-8th grade reading level, avoid idioms)
- Accessible pattern examples for buttons, links, errors, 
  form labels

See also the accessibility guidelines reference in the **design-content-guidelines** skill
for the comprehensive content accessibility guide.

## What to check

- Tab through all elements — logical order per 
  **design-accessibility** skill?
- All images have appropriate alt text (informational = 
  descriptive, decorative = empty, icon buttons = aria-label)?
- All form fields have visible Label component?
  (https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-label--docs)
- Color never the only indicator? (Pair with text/icon 
  per content-guidelines multi-modal guidelines and 
  **design-accessibility** skill)
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
