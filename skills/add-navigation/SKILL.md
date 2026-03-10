---
name: add-navigation
description: "MUST be used whenever adding a navigation header or top nav bar to a Dune app. Do NOT manually write a navigation header from scratch — this skill scaffolds the Navigation component, wires active state, and handles theme-aware logos. Triggers: navigation, nav bar, header, top nav, sticky header, nav items, active page, nav links, add navigation, navigation component."
allowed-tools: Read, Glob, Grep, Edit, Write
metadata:
  argument-hint: "[nav-item-ids e.g. home dashboard settings]"
---

# Add Navigation

Add a sticky header `Navigation` component to this Dune app with nav items, active state, and optional theme toggle.

Nav items: **$ARGUMENTS**

## Background

The Navigation component is a pure, dependency-injected sticky header. It owns no state and makes no routing assumptions. The parent controls:
- Which item is active (`activeId`)
- What happens on click (`onNavigate`)
- Logo content (swap by theme in the parent)
- Optional right-side slot (`renderThemeToggle`)


---

## Step 1 — Understand the app

Read these files before touching anything:

- `src/App.tsx` (or the main layout file) — understand current structure and routing
- `package.json` — detect package manager and existing deps (look for react-router-dom, etc.)

---

## Step 2 — Create the component files

Create `src/components/navigation/` and write these three files:

### `src/components/navigation/types.ts`

```ts
import type { ReactNode } from "react";

export interface NavItem {
  id: string;
  label: string;
}

export interface NavigationProps {
  items: NavItem[];
  activeId: string;
  onNavigate: (id: string) => void;
  logo: ReactNode;
  onLogoClick?: () => void;
  renderThemeToggle?: () => ReactNode;
  className?: string;
}
```

### `src/components/navigation/navigation.tsx`

```tsx
import type { NavigationProps } from "./types";

function cn(...classes: (string | undefined | false)[]): string {
  return classes.filter(Boolean).join(" ");
}

export function Navigation({
  items,
  activeId,
  onNavigate,
  logo,
  onLogoClick,
  renderThemeToggle,
  className,
}: NavigationProps) {
  return (
    <header
      className={cn("sticky top-0 z-50 border-b border-border bg-background", className)}
      role="banner"
    >
      <div className="flex h-14 items-center justify-between px-5">
        <div className="flex items-center gap-8">
          <button
            type="button"
            onClick={onLogoClick}
            className="flex items-center justify-center"
            aria-label="Go to home"
          >
            {logo}
          </button>

          <nav className="flex items-center gap-2" aria-label="Main navigation">
            {items.map((item) => {
              const isActive = activeId === item.id;
              return (
                <button
                  key={item.id}
                  type="button"
                  onClick={() => onNavigate(item.id)}
                  className={cn(
                    "relative px-4 py-2 text-base font-medium transition-colors",
                    isActive
                      ? "text-foreground"
                      : "text-muted-foreground hover:text-foreground",
                  )}
                  aria-current={isActive ? "page" : undefined}
                >
                  {item.label}
                  {isActive && (
                    <span
                      className="absolute bottom-0 left-4 right-4 h-0.5 bg-foreground"
                      aria-hidden
                    />
                  )}
                </button>
              );
            })}
          </nav>
        </div>

        {renderThemeToggle && (
          <div className="flex items-center gap-3">{renderThemeToggle()}</div>
        )}
      </div>
    </header>
  );
}
```

### `src/components/navigation/index.ts`

```ts
export { Navigation } from "./navigation";
export type { NavigationProps, NavItem } from "./types";
```

---

## Step 3 — Define nav items

Based on `$ARGUMENTS` (or the app's existing pages if arguments are empty), create a typed `NavItem[]` array. Use the app's existing page/route identifiers as `id` values:

```ts
import type { NavItem } from "@/components/navigation";

const NAV_ITEMS: NavItem[] = [
  { id: "home", label: "Home" },
  // Add one entry per argument / page
];
```

---

## Step 4 — Wire active state in the parent

Find the main layout component and add the Navigation. The parent owns `activeId`:

```tsx
import { useState } from "react";
import { Navigation } from "@/components/navigation";
import logo from "@/assets/logo.svg";

export function App() {
  const [activeId, setActiveId] = useState(NAV_ITEMS[0].id);

  return (
    <div className="flex min-h-screen flex-col">
      <Navigation
        items={NAV_ITEMS}
        activeId={activeId}
        onNavigate={setActiveId}
        logo={<img src={logo} alt="App" className="h-8 w-auto" />}
        onLogoClick={() => setActiveId(NAV_ITEMS[0].id)}
      />
      <main className="flex flex-1 flex-col">{/* page content */}</main>
    </div>
  );
}
```

---

## Step 5 — Add theme toggle (if the app has dark/light mode)

Check if the app uses `useDarkMode` or a theme hook. If yes, add `renderThemeToggle` and make the logo theme-aware:

```tsx
import logoLight from "@/assets/logo-light.svg";
import logoDark from "@/assets/logo-dark.svg";

// Inside the component:
const { isDark, toggle } = useDarkMode(); // or whatever hook the app uses

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
  onLogoClick={() => setActiveId(NAV_ITEMS[0].id)}
  renderThemeToggle={() => (
    <button
      type="button"
      onClick={toggle}
      aria-label={isDark ? "Switch to light mode" : "Switch to dark mode"}
    >
      {isDark ? "🌙" : "☀️"}
    </button>
  )}
/>
```

---

## Step 6 — Integrate with a router (if react-router-dom is present)

If `react-router-dom` is installed, sync `activeId` with the current route instead of using `useState`:

```tsx
import { useLocation, useNavigate } from "react-router-dom";

const location = useLocation();
const navigate = useNavigate();

// Derive activeId from the path — adjust to match the app's route structure
const activeId = location.pathname.split("/")[1] || NAV_ITEMS[0].id;

<Navigation
  items={NAV_ITEMS}
  activeId={activeId}
  onNavigate={(id) => navigate(`/${id === NAV_ITEMS[0].id ? "" : id}`)}
  logo={/* ... */}
/>
```

---

## Done

The app now has a sticky navigation header with active state highlighting for **$ARGUMENTS**. Accessibility is built in: `aria-current="page"` on the active item, `aria-label` on the logo button and nav element.
