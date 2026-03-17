---
name: use-aura-design-system
description: "MUST be used whenever implementing, migrating to, or reviewing usage of the Aura Design System in a Dune or React app. Use it to check Aura primitives and patterns docs, consult Storybook, install `@cognite/aura`, and replace local shadcn/ui components only when Aura publishes an equivalent package export. Triggers: Aura, Aura Design System, @cognite/aura, Aura primitives, Aura patterns, Aura storybook, replace shadcn, shadcn migration."
allowed-tools: Read, Glob, Grep, Edit, Write, Bash
---

# Use Aura Design System

Use Aura as the source of truth for primitives, patterns, and migration decisions.

## Basics

Before changing anything, read:

- `package.json` - detect package manager and existing UI dependencies
- The app entry file or global stylesheet - confirm where a one-time CSS import belongs
- The target component and any local `shadcn/ui` implementation it currently uses

If the app already hides local UI components behind a wrapper, prefer swapping the internals before changing call sites.

## Aura references

Use these sources:

- Primitives docs: `https://cognitedata.github.io/aura/primitives`
- Patterns docs: `https://cognitedata.github.io/aura/patterns`
- Storybook: `https://cognitedata.github.io/aura/storybook/`
- npm package: `https://www.npmjs.com/package/@cognite/aura`

Rules:

- Use docs for process and composition guidance.
- Use Storybook for concrete states, variants, and visual behavior.
- Use npm as the source of truth for what is actually installable.
- If a direct docs route returns a 404, open `https://cognitedata.github.io/aura/` and navigate to the section from the app shell.

## Package basics

Install Aura with the app's package manager, import the styles once, and import components from `@cognite/aura/components`.

## Replacement rule

- Prefer Aura props, composition, and shipped styling over reapplying old shadcn class stacks.
- Verify that the published package exports the target component before replacing a local implementation.
- Do not assume every Storybook example is published to npm yet.
- If Aura publishes an equivalent component, replace the local shadcn implementation or wrap Aura behind the existing local API.
- If Aura does not publish it, keep the local implementation and explain the gap.

## Browser auth

If browser-based verification is needed in a protected app, do the first run in a headed browser and capture auth state or a token before relying on headless Playwright. Treat redirects, blank pages, and `401`/`403` responses as auth setup problems first.

## Verify

Run the relevant build or test command and compare the final UI against the Aura docs and Storybook states you used as references.
