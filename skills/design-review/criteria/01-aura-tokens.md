# Criterion 1: Aura Token Consistency — Check Details

## Cross-reference

See the **design-aura-tokens** skill for rules.
Storybook Foundations for correct token values.

## What to check

- Any hardcoded hex/rgb values in className or style props?
- Any Tailwind default colors (gray-500, red-100, bg-white)?
  These should be Aura semantic tokens (text-muted-foreground, 
  bg-destructive, bg-background). See **design-aura-tokens** skill 
  common mistakes table.
- All components from Aura library? Check against 
  **design-aura-components** skill component inventory.
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
