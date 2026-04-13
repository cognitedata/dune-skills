---
name: use-topbar
description: >-
  Wires the Aura Topbar (@aura/topbar) into a Dune or Fusion app as the single
  top navigation bar with breadcrumbs, optional middle metadata, right-slot
  actions, and a built-in dark mode toggle. Use when adding a topbar, top
  navigation bar, app chrome, navigation header, breadcrumbs, or dark mode to a
  Dune app. Also use when replacing or deprecating the add-navigation skill, or
  when the user mentions topbar, Topbar, use-topbar, top nav, app header, or
  dark mode toggle in a Dune or Fusion app context.
allowed-tools: Read, Glob, Grep, Write, Edit, Shell
---

# Use Topbar

Wires the Aura `@aura/topbar` component into a Dune app as the single, compliant top navigation bar. Replaces the deprecated `add-navigation` skill (`@cognite/dune-industrial-components/navigation`).

**Canonical Storybook:** [Primitives — Topbar](https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-topbar--docs)
**Full rules:** [topbar-rules.md](topbar-rules.md)

---

## Step 1 — Pre-flight: read the app

Before asking any questions, read:

- `package.json` — package manager, existing UI deps, existing `@aura/topbar`
- `src/App.tsx` (or main layout file) — routing setup, existing dark mode hook/context
- Any Dune/Fusion app config (`app.config.ts`, `fusion.config.ts`, manifest files) — `displayName`, `name`, `icon` fields

Note what you find. If app name, icon, or a dark mode hook already exists, apply those as defaults and skip the corresponding interview questions. State what was inferred.

---

## Step 2 — Configuration interview (mandatory)

Run this interview **before implementing anything**. Ask **one question at a time** and wait for the answer. Skip questions already answered by Step 1.

| Step | Topic | Question |
|------|--------|----------|
| A | Shell ownership | Where will the Topbar be mounted — in the **shared shell/layout** or **only inside this app's** layout? |
| B | App display name | What **application display name** should appear as the **first breadcrumb** segment? |
| C | App icon | Is the **application icon** already defined by Aura/shell/Dune for this app, or do we need to configure one? |
| D | Object concept | Does this app have routes where a **single domain object** is the main focus (detail/editor/viewer)? |
| E | Object label | *(if D = yes)* What **user-visible label** identifies that object in the last breadcrumb segment? |
| F | Object menu | *(if D = yes)* Should the object name open a **dropdown** with object-level actions? |
| G | Menu items | *(if F = yes)* What actions should appear (label + handler)? |
| H | Middle metadata | Should the **middle** region show read-only metadata (e.g. last updated, created by)? |
| I | Right slot content | Besides system actions and the dark mode toggle, what belongs in the **right slot** (nothing / tabs / avatars / secondary buttons)? |
| J | Primary CTA | Is there a **single** primary action button for the right slot (Aura Default, small)? |
| K | System actions | Should **agent**, **notifications**, or **share** be hidden for this app? |
| L | Topbar omission | Are there routes (login, fullscreen modal) where the Topbar should **not** render? |

**Closing:** Summarize left / middle / right / system actions / omissions in five bullets or fewer, then proceed.

---

## Step 3 — Install

Check `package.json` for `@aura/topbar`. If absent:

```bash
pnpm dlx shadcn@latest add @aura/topbar
```

Also confirm `tailwind.config` has `darkMode: 'class'`. If missing, add it.

---

## Step 4 — Dark mode hook

Always implement dark mode. Check for an existing theme hook first:

- Search for `useDarkMode`, `useTheme`, `useColorScheme`, or a `ThemeProvider` in `src/`
- If found, wire into it and skip creating a new hook

If no theme system exists, create `src/hooks/use-dark-mode.ts`:

```ts
import { useEffect, useState } from 'react';

export function useDarkMode() {
  const [isDark, setIsDark] = useState(() => {
    const stored = localStorage.getItem('theme');
    if (stored) return stored === 'dark';
    return window.matchMedia('(prefers-color-scheme: dark)').matches;
  });

  useEffect(() => {
    document.documentElement.classList.toggle('dark', isDark);
    localStorage.setItem('theme', isDark ? 'dark' : 'light');
  }, [isDark]);

  return { isDark, toggle: () => setIsDark((v) => !v) };
}
```

Apply the initial class on page load by adding to the app entry point (`main.tsx` / `index.tsx`):

```ts
const stored = localStorage.getItem('theme');
const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
if (stored === 'dark' || (!stored && prefersDark)) {
  document.documentElement.classList.add('dark');
}
```

---

## Step 5 — Implement the Topbar

**Always check Storybook for exact prop names before writing code.** The names below are illustrative — verify against the current `@aura/topbar` package.

```tsx
import { Topbar } from '@aura/topbar';
import { Breadcrumb, BreadcrumbItem } from '@aura/topbar'; // adjust to actual exports
import { Button } from '@cognite/aura/components';
import { SunIcon, MoonIcon } from '@cognite/aura/icons'; // adjust to actual icon exports
import { useDarkMode } from '@/hooks/use-dark-mode';

export function AppShell({ children }: { children: React.ReactNode }) {
  const { isDark, toggle } = useDarkMode();

  return (
    <>
      <Topbar
        // Left — required
        applicationIcon={<AuraAppIcon name="..." />}
        breadcrumbs={
          <Breadcrumb>
            <BreadcrumbItem label="Application Name" />
            {/* Add object segment only when a domain object is in focus */}
          </Breadcrumb>
        }

        // Middle — optional metadata; omit or pass null when unused
        middleContent={null}

        // Right flexible slot
        trailingSlot={
          <>
            {/* App controls (tabs, avatars, secondary buttons) go here */}

            {/* Dark mode toggle — always present, always leftmost of app controls */}
            <Button
              variant="ghost"
              size="sm"
              onClick={toggle}
              aria-label={isDark ? 'Switch to light mode' : 'Switch to dark mode'}
            >
              {isDark ? <SunIcon /> : <MoonIcon />}
            </Button>

            {/* Primary CTA — at most one, rightmost in slot */}
            {/* <Button variant="default" size="sm">Primary action</Button> */}
          </>
        }

        // System actions — fixed order; apps may hide individual ones
        systemActions={{
          agent: { visible: true },
          notifications: { visible: true },
          share: { visible: true },
        }}
      />
      <main>{children}</main>
    </>
  );
}
```

**Layout wrapper:** The parent element must allow the Topbar to be full-width and sticky. Typical pattern:

```tsx
<div className="flex min-h-screen flex-col">
  <AppShell>
    {/* page content */}
  </AppShell>
</div>
```

---

## Step 6 — Region rules (quick reference)

| Region | Rule |
|--------|------|
| Left | Icon (far left) → breadcrumbs. App name always in first segment. Object name in last segment only when a domain object is in focus. |
| Middle | Non-interactive metadata only. Leave `null` when unused — do not collapse layout. |
| Right flexible slot | Dark mode toggle (always) → other app controls → primary CTA (one `variant="default"`, `size="sm"`, rightmost). |
| System actions | Fixed order: agent → notifications → share. No style or behavior overrides. Apps may hide; never reorder. |
| Sizing | All interactive elements use `size="sm"`. |

---

## Step 7 — Compliance checklist

Verify before finishing:

- [ ] Exactly **one** Topbar per page — no duplicate headers or embedded Topbars
- [ ] Left section: icon → breadcrumbs in correct order
- [ ] App name present in first breadcrumb segment
- [ ] Object name in last breadcrumb segment **only** when a domain object is in focus
- [ ] Dark mode toggle present in right slot; `useDarkMode` hook wired
- [ ] At most **one** `variant="default"` button in the right slot
- [ ] All Topbar controls use `size="sm"`
- [ ] System actions retain Aura styling — no custom overrides
- [ ] No `style={{}}` or `className` overrides on Topbar or its children
- [ ] Aura semantic tokens used throughout (no hardcoded colors)
- [ ] `tailwind.config` has `darkMode: 'class'`
- [ ] `add-navigation` (`@cognite/dune-industrial-components/navigation`) removed if previously present

---

## Additional resources

- Full Topbar architecture rules and configuration model: [topbar-rules.md](topbar-rules.md)
- Aura Topbar Storybook: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-topbar--docs
- Aura Breadcrumb Storybook: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-breadcrumb--docs
- Aura Button Storybook: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-button--docs
- Aura colors / dark mode tokens: https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-colors--docs
