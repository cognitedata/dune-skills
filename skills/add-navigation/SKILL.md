---
name: add-navigation
description: "Wires the Navigation component from @cognite/dune-industrial-components/navigation into a Dune app. Use when adding a navigation header, top nav bar, sticky header, nav items, active page, nav links, navigation component, theme toggle, or dark mode support to a Dune app."
allowed-tools: Read, Glob, Grep, Write, Edit
metadata:
  argument-hint: "[nav-item-ids e.g. home dashboard settings]"
---

# Add Navigation

Wire `Navigation` from `@cognite/dune-industrial-components/navigation` into this Dune app.

Nav items: **$ARGUMENTS**

The component is fully controlled — the parent owns `activeId` and `onNavigate`. No state or routing assumptions live inside it.

The `logo` prop is optional and defaults to the Cognite vertical logo. It uses `currentColor` so it automatically renders black in light mode and white in dark mode.

## Step 1 — Check dependencies

First, check if `@cognite/dune-industrial-components` is installed by looking at `package.json`. If not installed, run:

```bash
pnpm install "github:cognitedata/dune-industrial-components#semver:*"
```

## Step 2 — Read the app

Read `src/App.tsx` (or the main layout file) and `package.json` before changing anything. Note:
- Existing routing (`react-router-dom`?)
- Existing dark-mode / theme hook
- Where page content currently lives

## Step 3 — Define nav items

Create a typed `NavItem[]` array using `$ARGUMENTS` (fall back to the app's existing pages/routes if empty):

```ts
import type { NavItem } from '@cognite/dune-industrial-components/navigation';

const NAV_ITEMS: NavItem[] = [
  { id: 'home', label: 'Home' },
  // one entry per argument / page
];
```

## Step 4 — Wire into the layout

The Navigation uses the default Cognite logo and includes a theme toggle by default.

### No router (useState)

```tsx
import { useState } from 'react';
import { Navigation } from '@cognite/dune-industrial-components/navigation';

export function App() {
  const [activeId, setActiveId] = useState(NAV_ITEMS[0].id);
  
  // If the app has dark mode, use the existing hook
  const { isDark, toggle } = useDarkMode(); // Adjust to match app's theme hook

  return (
    <div className="flex min-h-screen flex-col">
      <Navigation
        items={NAV_ITEMS}
        activeId={activeId}
        onNavigate={setActiveId}
        renderThemeToggle={() => (
          <button
            type="button"
            onClick={toggle}
            aria-label={isDark ? 'Switch to light mode' : 'Switch to dark mode'}
          >
            {isDark ? '🌙' : '☀️'}
          </button>
        )}
      />
      <main className="flex flex-1 flex-col">{/* page content */}</main>
    </div>
  );
}
```

### With react-router-dom

```tsx
import { useLocation, useNavigate } from 'react-router-dom';
import { Navigation } from '@cognite/dune-industrial-components/navigation';

export function App() {
  const location = useLocation();
  const navigate = useNavigate();
  
  // If the app has dark mode, use the existing hook
  const { isDark, toggle } = useDarkMode(); // Adjust to match app's theme hook

  // Derive activeId from path — adjust to match the app's route structure
  const activeId = location.pathname.split('/')[1] || NAV_ITEMS[0].id;

  return (
    <div className="flex min-h-screen flex-col">
      <Navigation
        items={NAV_ITEMS}
        activeId={activeId}
        onNavigate={(id) => navigate(`/${id === NAV_ITEMS[0].id ? '' : id}`)}
        renderThemeToggle={() => (
          <button
            type="button"
            onClick={toggle}
            aria-label={isDark ? 'Switch to light mode' : 'Switch to dark mode'}
          >
            {isDark ? '🌙' : '☀️'}
          </button>
        )}
      />
      <main className="flex flex-1 flex-col">{/* page content */}</main>
    </div>
  );
}
```

## Optional — Custom logo

By default, the Navigation uses the Cognite logo which automatically adapts to light/dark themes. Only add a custom logo if needed.

To use a custom logo, pass the `logo` prop. If the app has dark/light mode, swap the image based on the theme hook:

```tsx
import { Navigation } from '@cognite/dune-industrial-components/navigation';
import logoLight from '@/assets/logo-light.svg';
import logoDark from '@/assets/logo-dark.svg';

// Inside the component (whatever hook the app uses):
const { isDark } = useDarkMode();

<Navigation
  items={NAV_ITEMS}
  activeId={activeId}
  onNavigate={setActiveId}
  logo={
    <img
      src={isDark ? logoDark : logoLight}
      alt="App"
      className="h-8 w-auto"
    />
  }
  renderThemeToggle={() => (
    <button
      type="button"
      onClick={toggle}
      aria-label={isDark ? 'Switch to light mode' : 'Switch to dark mode'}
    >
      {isDark ? '🌙' : '☀️'}
    </button>
  )}
/>
```

## Done

The app now has a sticky navigation header with:
- Default Cognite logo that adapts to light/dark themes
- Theme toggle button (if dark mode hook exists)
- `aria-current="page"` on the active item
- `aria-label` attributes on the logo button and `<nav>` element

> **Package**: `@cognite/dune-industrial-components/navigation`
> **Exports**: `Navigation`, `CogniteLogo`, `NavItem`, `NavigationProps`
