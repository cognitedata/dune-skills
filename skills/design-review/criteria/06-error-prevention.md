# Criterion 6: Error Prevention — Check Details

## Cross-reference

See the **design-content-guidelines** skill for confirmation dialog content:
- Title: [Action verb] [object]?
- Body: consequences + reversibility
- Confirm button: specific action verb (never Yes/No)
- Tone: serious, transparent, respectful

See the **design-aura-components** skill for confirmation components:
- Alert Dialog: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-alert-dialog--docs
- Dialog: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-dialog--docs

See the **design-error-validation** skill for confirmation patterns.

## What to check

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
