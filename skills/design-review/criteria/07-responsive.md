# Criterion 7: Responsive Design — Check Details

## Cross-reference

See the **design-layout-patterns** skill for responsive behavior per layout.
See the **design-aura-tokens** skill — Storybook Layout foundation for 
breakpoints and container queries:
  https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-layout--docs
See the **design-aura-components** skill:
- Sidebar: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-sidebar--docs
  (check collapse behavior)
- Drawer: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-drawer--docs
  (mobile navigation alternative)

## What to check

- Test at 1440px, 1024px, 768px, 375px widths?
- No horizontal scrolling at any viewport?
- Touch targets 44px+ on mobile?
- Hover-dependent features have touch alternatives?
- Sidebar collapses/hides on mobile?
- Text readable without zooming on mobile?
- Navigation adapts for smaller screens?
- Layout follows responsive behavior defined in 
  **design-layout-patterns** skill for the chosen pattern?
- Flex children have `min-w-0` to prevent text/input overflow?
- No fixed `w-[Npx]` or `h-[Npx]` inside flex/grid parents?
- Input/Textarea elements use `w-full` inside containers?
- Scroll containers have `overflow-hidden` or `overflow-auto`?
- Elements with `whitespace-nowrap` also have 
  `overflow-hidden text-ellipsis`?
- VISUAL: No elements visually exceeding parent bounds at 
  any tested viewport?
