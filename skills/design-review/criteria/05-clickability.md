# Criterion 5: Clickability — Check Details

## Cross-reference

See the **design-aura-components** skill — all interactive elements 
should use Aura components:
- Button: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-button--docs
- Toggle: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-toggle--docs
- Switch: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-switch--docs
- Dropdown Menu: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-dropdown-menu--docs

## What to check

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
  **design-aura-components** skill input-group-sizing)
