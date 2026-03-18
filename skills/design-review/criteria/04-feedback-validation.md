# Criterion 4: Feedback & Validation — Check Details

## Cross-reference

See the **design-content-guidelines** skill for message patterns and 
tone adaptation:
- Error types: validation (inline), system (modal/banner), 
  blocking (full-screen), permission
- Error pattern: [What failed]. [Why/context]. [What to do.]
- Success pattern: [Action] [result/benefit]
- Tone adaptation: frustrated → empathetic, cautious → 
  transparent, confident → efficient
- What to avoid: technical codes, blame language, robotic 
  tone, dead ends, vague causes

See the **design-aura-components** skill for feedback components:
- Alert: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-alert--docs
- Banner: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-banner--docs
- Sonner Toast: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-sonner-toast--docs
- Helper Text: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-helper-text--docs
- Skeleton: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-skeleton--docs
- Progress: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-progress--docs

See the **design-error-validation** skill for implementation patterns.

## What to check

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
  (See **design-error-validation** skill message-patterns table)
- Required fields show "required" indicator 
  (asterisk or "(required)" in label)?
- Validation fires on blur AND on submit?
- Form submission is blocked when validation errors exist?
