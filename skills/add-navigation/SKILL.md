---
name: add-navigation
description: "Wires the Navigation component from @cognite/dune-industrial-components/navigation into a Dune app. Use when adding a navigation header, top nav bar, sticky header, nav items, active page, nav links, or navigation component to a Dune app."
allowed-tools: Read, Glob, Grep, Write, Edit
metadata:
  argument-hint: "[nav-item-ids e.g. home dashboard settings]"
---

# Add Navigation

Wire `Navigation` from `@cognite/dune-industrial-components/navigation` into this Dune app.

Nav items: **$ARGUMENTS**

The component is fully controlled — the parent owns `activeId` and `onNavigate`. No state or routing assumptions live inside it.

The `logo` prop is optional and defaults to the Cognite vertical logo. It uses `currentColor` so it automatically renders black in light mode and white in dark mode.

## Step 1 — Read the app

Read `src/App.tsx` (or the main layout file) and `package.json` before changing anything. Note:
- Existing routing (`react-router-dom`?)
- Existing dark-mode / theme hook
- Where page content currently lives

## Step 2 — Define nav items

Create a typed `NavItem[]` array using `$ARGUMENTS` (fall back to the app's existing pages/routes if empty):

```ts
import type { NavItem } from '@cognite/dune-industrial-components/navigation';

const NAV_ITEMS: NavItem[] = [
  { id: 'home', label: 'Home' },
  // one entry per argument / page
];
```

## Step 3 — Wire into the layout

### No router (useState)

```tsx
import { useState } from 'react';
import { Navigation } from '@cognite/dune-industrial-components/navigation';

export function App() {
  const [activeId, setActiveId] = useState(NAV_ITEMS[0].id);

  return (
    <div className="flex min-h-screen flex-col">
      <Navigation
        items={NAV_ITEMS}
        activeId={activeId}
        onNavigate={setActiveId}
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

  // Derive activeId from path — adjust to match the app's route structure
  const activeId = location.pathname.split('/')[1] || NAV_ITEMS[0].id;

  return (
    <div className="flex min-h-screen flex-col">
      <Navigation
        items={NAV_ITEMS}
        activeId={activeId}
        onNavigate={(id) => navigate(`/${id === NAV_ITEMS[0].id ? '' : id}`)}
      />
      <main className="flex flex-1 flex-col">{/* page content */}</main>
    </div>
  );
}
```

## Step 4 — Custom logo (optional)

Skip this step if the Cognite default logo is appropriate.

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
/>
```

## Step 5 — Theme toggle (if the app has dark/light mode)

If there is a dark-mode hook, pass `renderThemeToggle` to add a toggle button to the right side of the header:

```tsx
const { isDark, toggle } = useDarkMode();

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
```

## Done

The app now has a sticky navigation header with `aria-current="page"` on the active item and `aria-label` attributes on the logo button and `<nav>` element.

> **Package**: `@cognite/dune-industrial-components/navigation`
> **Exports**: `Navigation`, `CogniteLogo`, `NavItem`, `NavigationProps`
