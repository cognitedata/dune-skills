---
name: use-topbar
description: >-
  Wires the Aura Topbar (@aura/topbar) into a Dune or Fusion app as the single
  top navigation bar with breadcrumbs, optional left inline metadata after the
  breadcrumb, optional center Tabs or Segmented control (small), and a right
  utility strip (Share, notifications, theme menu, Atlas, user Avatar) with
  per-control visibility. Use when adding a topbar, top navigation bar, app
  chrome, navigation header, breadcrumbs, or theme switching to a Dune app.
  Also use when replacing or deprecating the add-navigation skill, or when the
  user mentions topbar, Topbar, top nav, app header, or theme switcher in a
  Dune or Fusion app context. Also use when the user is starting, building,
  creating, or setting up a new Dune application, Fusion app, or Dune-based
  project — including any mention of "new Dune app", "Dune application",
  "building a Dune app", "starting a Fusion app", or any request to scaffold,
  wire up, or begin work on a Dune or Fusion application.
allowed-tools: Read, Glob, Grep, Write, Edit, Shell
---

# Topbar

`@aura/topbar` is the single, compliant top navigation bar for every authenticated Dune/Fusion app. It is a **shadcn registry component**, not an npm package:

```bash
pnpm dlx shadcn@latest add @aura/topbar
```

This requires `components.json` with the `@aura` registry configured. See Step 3 for the full decision tree.

Replaces the deprecated `add-navigation` skill (`@cognite/dune-industrial-components/navigation`).

**Canonical Storybook:** [Primitives — Topbar](https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-topbar--docs)
**Full rules:** [RULES.md](RULES.md)

---

> ### Non-negotiables
>
> See [RULES.md §1, §11, §12](RULES.md) for the full rule set. In short: exactly one Topbar per page, composed only from `@aura/topbar` primitives, installed via the shadcn CLI only. If installation fails, surface the blocker — never build a custom header fallback.

---

## Step 0 — Auto-prompt hook setup

Ensures the topbar configuration interview is automatically triggered at the start of every future Cursor session in this project.

1. Check whether `.cursor/hooks.json` exists in the project root.
2. If it does not exist, create it:

```json
{
  "version": 1,
  "hooks": {
    "sessionStart": [
      {
        "type": "prompt",
        "prompt": "This is a Dune/Fusion application. Before implementing any UI or navigation work, read and follow the topbar skill and run the topbar configuration interview with the user."
      }
    ]
  }
}
```

3. If `.cursor/hooks.json` already exists but does not contain a `sessionStart` entry for the topbar, merge the entry above — do not overwrite unrelated hooks.
4. Confirm: "I've added a session hook so the topbar interview will be prompted automatically every time you open this project in Cursor."

Proceed to Step 1 immediately after.

---

## Step 1 — Pre-flight: read the app

Before asking any questions, read:

- `package.json` — package manager, existing UI deps, existing `@aura/topbar`
- `src/App.tsx` (or main layout file) — routing setup, existing dark mode hook/context
- Any Dune/Fusion app config (`app.config.ts`, `fusion.config.ts`, manifest fields) — `displayName`, `name`, and any **app mark** / icon / branding used for the left `Avatar`

Note what you find. If app name, branding, or a theme hook already exists, apply those as defaults and skip the corresponding interview questions. State what was inferred.

---

## Step 2 — Configuration interview (mandatory)

**Complete this interview before writing any implementation code.** Do not skip, shorten, or defer it. If completing it mid-task feels disruptive, pause the task, run the interview, then resume.

Ask **one question at a time** and wait for the answer. Skip only questions that Step 1 already answered definitively.

### Topbar layout reference

Use this diagram to orient yourself and the user throughout the interview:

```
┌──────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ [App Avatar] [App] > [Object▾] metadata…  │  (optional) Tabs or Segmented — sm  │ Share Bell Theme Atlas User │
│ ←── Left: fjord Avatar + breadcrumb + metadata (left-aligned, not centered) ──→ │ ←── Middle ──→ │ ←── Right strip (fixed order) ──→ │
└──────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

**Left section — breadcrumb states:**

```
No object open:         [App Avatar]  My App
Object open:            [App Avatar]  My App  >  Root Cause Analysis ▾
With inline metadata:   [App Avatar]  My App  >  Root Cause Analysis ▾   Updated 3 hours ago
```

- Metadata always continues **on the left**, immediately after the breadcrumb — it is **never** centered in the Topbar.
- App name clicking navigates to the app's home/root route — but only when an object is open. If no object is open, the app name is not a link.
- Object name is clickable only when it acts as a dropdown trigger (▾). Each breadcrumb segment is an interactive link that navigates to its route.

**Middle section — Tabs or Segmented control (optional):**

```
Tabs (routes):          [Overview]  [Well analysis]  [Settings]
Segmented (modes):      [Canvas view]  [Code view]
```

- **Tabs** — mutually exclusive **page-level** views (routes).
- **Segmented control** — **mode** or layout switching (e.g. canvas vs code) when that fits better than route tabs.
- **Always `size="sm"`** (or Aura’s equivalent **small** size for these primitives).
- Omit entirely if the app has no global navigation or only one view.
- **Do not** put app-specific **primary** actions here — those belong **below** the Topbar.

**Right section — utility strip (fixed order when each control is shown):**

```
Share (ghost, sm)   Notifications (ghost, sm)   Theme (ghost, sm)   Atlas (secondary, sm)   User Avatar (sm)
     optional              optional                  optional              optional              optional*
```

\*Theme and user Avatar are **typically on**; turn off only when the API and product policy allow.

**Theme control:**

- **Light mode** → **sun** icon on the trigger; **dark mode** → **moon** icon.
- Clicking opens a **Menu** with **Light mode** and **Dark mode** rows; a **checkmark** shows the current selection; only **one** row is active at a time.

If Storybook exposes extra right-slot entries (e.g. legacy agent), follow the **current** Aura API and [RULES.md §3.3](RULES.md).

---

### Left section

**Q1 — Application mark (Avatar)**

> "At the far left we use a small Aura Avatar in the fjord colorway for the app mark. Does your app already have branding or an image in config, or should we use the default fjord Avatar treatment from Aura?"

- Prefer assets from Dune/Fusion app config when present.
- Compose with Aura **`Avatar`**, **`size="small"`**, **`fjord`** (exact props from Storybook).

**Q2 — App name**

> "What is the name of your application? It will appear as the first breadcrumb and always be visible."

- If already defined in app config (`displayName`, `name`), apply it and skip.

**Q3 — App structure and breadcrumbs**

> "Referring to the breadcrumb states above — which best describes your app: single app name only (no objects), or does the app let users open specific named items like a canvas, report, or document?"

- **No objects** → only the app name appears in the breadcrumb. The app name is not a link (there is nowhere to navigate back to).
- **Named objects** → app name always visible; object name added as the last segment only when an object is open. If yes: "What do you call these items? (e.g. Canvas, Report, Dashboard)"

**Breadcrumb interactivity rules (non-negotiable):**
- All breadcrumb segments are **interactive links** — they must navigate to their corresponding route, not be plain text.
- When an object is open, clicking the **app name** navigates back to the app's home/root (e.g. the object list or landing page).
- When no object is open, the app name segment is **not** a link (it is the current location).
- Do not render breadcrumbs as static/non-interactive text.

**Q4 — Object actions dropdown** _(ask only if Q3 identified named objects)_

> "When a user has a specific [object type] open, would you like a dropdown menu on its name in the breadcrumb for object-level actions — like rename, duplicate, export, or delete?"

- This dropdown appears **only on the object name** (the last breadcrumb segment), and **only when an object is currently open**.
- **There is no dropdown on the app name.** If users need app-level settings (e.g. manage permissions, configure defaults), place entry points in the **content area below the Topbar** (or another approved pattern), not inside the object dropdown.
- All actions in the object dropdown must apply **only to the currently open object** — do not mix in app-level or global actions.
- Examples: "Rename this canvas", "Duplicate this report", "Export this document", "Delete this item".
- If yes: "What object-specific actions should appear in the menu?"

---

### Left section — inline metadata

**Q5 — Inline metadata** _(optional)_

> "Would you like a short status string directly after the breadcrumb on the **left** — things like 'Updated 3 hours ago' or 'Read-only'? It stays in the left cluster with the breadcrumb, never centered in the bar."

- If yes: "What text should appear there?"
- **String only** — no links, icons, or interactive elements.
- Omit entirely if unused — do not add a placeholder.
- Typical use: last-modified time, read-only state, a status label tied to the current object or page.

---

### Middle section — navigation

**Q6 — Global navigation (Tabs or Segmented)** _(optional)_

> "Does your app need global navigation in the center of the Topbar — either **Tabs** for mutually exclusive pages/routes, or a **Segmented control** for modes like canvas vs code?"

- **Tabs** — primary app sections as routes (e.g. Overview → `/overview`, Settings → `/settings`).
- **Segmented control** — switching **views or modes** within the app without changing the top-level route model, when that fits better.
- **Always small** size to match the rest of the Topbar.
- **Never use a sidebar for primary navigation.** If the app needs additional internal navigation beyond this slot, it must live within the content area — not as a sidebar.
- Only include controls that are relevant globally. Page-specific sub-navigation belongs in the content area.
- If yes: "Which pattern (Tabs vs Segmented), what are the labels, and where does each choice lead?"
- **Default:** leave the center empty and keep **primary actions below the Topbar**.

---

### Right section — utility strip

**Q7 — Primary actions in the Topbar** _(reframe as guidance, not a button inventory)_

> "We no longer place app-specific primary CTAs in the Topbar — those should live in the content area below it. Are you comfortable leaving the Topbar without '+ Create' / 'Export' style buttons, or is there a rare, truly app-wide control you still need next to the utility icons?"

- **Default:** no extra action buttons in the Topbar shell.
- If something is proposed, apply the test: does it apply to the **entire app on every screen**? If not, it belongs **below** the Topbar.
- When in doubt, omit it from the Topbar.

**Q8 — Right-strip controls** _(ask each sub-question separately)_

> "The right side is a fixed-order strip; you can turn each control on or off depending on capabilities. I'll ask about each one."

Ask each separately (in this order for consistency with the bar):

- **Share:** "Do users need share? Should the Share icon (ghost, small) appear?"
- **Notifications:** "Does this app surface notifications? Should the bell appear?"
- **Theme menu:** "Should users switch light/dark theme from the Topbar (sun/moon trigger + menu with checkmarked Light/Dark rows)?"
- **Atlas:** "Does this app use Atlas? Should the secondary Atlas button (leading icon + label) appear?"
- **User Avatar:** "Should the signed-in user Avatar appear on the far right?"

Fixed order when visible (left → right): **Share → Notifications → Theme → Atlas → user Avatar**.

Apps **must not** reorder these items; styling follows Aura. If Aura documents additional optional controls, align with Storybook.

---

### Omissions

**Q9 — Excluded routes**

> "Are there any screens where the top bar should NOT appear? Common exceptions: login/auth screens, fullscreen flows, onboarding. Default is to show it everywhere."

---

**Closing:** Before implementing, summarize the configuration in five bullets or fewer: left (fjord `Avatar` + breadcrumb pattern + inline metadata if any), middle (Tabs, Segmented control, or none), primary actions (confirm they live **below** the Topbar), right strip (which of Share / Notifications / Theme / Atlas / Avatar are on), excluded routes. Then proceed to Step 3.

---

## Step 3 — Install

**Installation is mandatory.** If `@aura/topbar` cannot be installed, stop and surface the blocker. Do not build a custom component or any workaround.

### 3a — Check if already installed

Check `package.json` for `@aura/topbar`. If present, skip to Step 3d.

### 3b — Determine install method

`@aura/topbar` is a **shadcn registry component** — not on npm. The only valid install path is the shadcn CLI (`pnpm dlx shadcn@latest add`). Do not use `npm install`, `pnpm add`, or `yarn add`.

Before running the install:

1. **Ensure `components.json` has the `@aura` registry.** If absent or missing the entry, add:

   ```json
   {
     "registries": {
       "@aura": "https://cognitedata.github.io/aura/r/{name}.json"
     }
   }
   ```

   If `components.json` does not exist at all, run `pnpm dlx shadcn@latest init` first, then add the entry.

2. **Detect the package manager:**
   - `pnpm-lock.yaml` → pnpm
   - `yarn.lock` → yarn
   - `package-lock.json` → npm

### 3c — Install

```bash
pnpm dlx shadcn@latest add @aura/topbar
```

> **If this fails**, stop. Tell the user exactly what failed and ask them to resolve the blocker. Do not proceed with a workaround.

### 3d — Tailwind check

Confirm `tailwind.config` has `darkMode: 'class'`. Add it if missing.

---

## Step 4 — Dark mode hook

Always implement theme switching (light / dark). Check for an existing theme system first:

- Search for `useDarkMode`, `useTheme`, `useColorScheme`, or a `ThemeProvider` in `src/`
- If found, wire into it and skip creating a new hook.

If none exists, create `src/hooks/use-theme-mode.ts` (or extend your existing hook) so the Topbar menu can **set** light or dark explicitly:

```ts
import { useEffect, useState } from 'react';

export type ThemeMode = 'light' | 'dark';

export function useThemeMode() {
  const [mode, setMode] = useState<ThemeMode>(() => {
    const stored = localStorage.getItem('theme');
    if (stored === 'dark' || stored === 'light') return stored;
    return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
  });

  useEffect(() => {
    const isDark = mode === 'dark';
    document.documentElement.classList.toggle('dark', isDark);
    localStorage.setItem('theme', isDark ? 'dark' : 'light');
  }, [mode]);

  return {
    mode,
    isDark: mode === 'dark',
    setTheme: (next: ThemeMode) => setMode(next),
  };
}
```

Apply the initial class on page load in `main.tsx` / `index.tsx`:

```ts
const stored = localStorage.getItem('theme');
const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
if (stored === 'dark' || (!stored && prefersDark)) {
  document.documentElement.classList.add('dark');
}
```

The Topbar **theme** trigger should open a **Menu** whose items call `setTheme('light')` and `setTheme('dark')` and show a **checkmark** on the active row.

---

## Step 5 — Implement the Topbar

**Always check Storybook for exact prop names before writing code.** The names below are illustrative — verify against the current `@aura/topbar` package.

```tsx
import { Topbar } from '@aura/topbar';
import { Breadcrumb, BreadcrumbItem } from '@aura/topbar'; // adjust to actual exports
// App + user Avatar: import from the Aura package / path Storybook documents for Topbar.
import { useThemeMode } from '@/hooks/use-theme-mode';

export function AppShell({ children }: { children: React.ReactNode }) {
  const { mode, setTheme } = useThemeMode();

  return (
    <>
      <Topbar
        // Left — application mark: Avatar small, fjord (verify Storybook props)
        applicationIcon={
          <Avatar size="small" colorway="fjord" src={appMarkSrc} alt="" />
        }
        breadcrumbs={
          <Breadcrumb>
            <BreadcrumbItem label="Application Name" href="/" />
            {/* <BreadcrumbItem label={objectName} ... dropdown ... /> */}
          </Breadcrumb>
        }

        // Inline metadata — optional string immediately after breadcrumb, left-aligned only
        // breadcrumbMetadata="Updated 3 hours ago"

        // Middle — optional Tabs (routes) OR Segmented control (modes); size small; omit if unused
        centerSlot={
          null
          // Example Tabs: <Tabs size="sm" ... />
          // Example Segmented: <SegmentedControl size="sm" ... />
        }

        // Right strip — fixed order when each is visible: share → notifications → theme → atlas → avatar
        // Theme: sun when light, moon when dark; Menu with Light mode / Dark mode + checkmark on active
        // Storybook may still call this darkMode or split props differently — map menu choice to setTheme('light'|'dark').
        trailingSlot={null}
        systemActions={{
          share: { visible: true },
          notifications: { visible: true },
          darkMode: {
            visible: true,
            mode, // 'light' | 'dark' — illustrative; use whatever resolvedTheme API Aura exposes
            onSelectLight: () => setTheme('light'),
            onSelectDark: () => setTheme('dark'),
          },
          atlas: { visible: true },
          avatar: { visible: true, src: userPhotoSrc, alt: userName },
        }}
      />
      <main>{children}</main>
    </>
  );
}
```

**Layout wrapper:** The parent element must allow the Topbar to be full-width and sticky:

```tsx
<div className="flex min-h-screen flex-col">
  <AppShell>
    {/* page content — primary actions for the current screen live here */}
  </AppShell>
</div>
```

---

## Step 6 — Compliance checklist

Verify before finishing (see [RULES.md §12](RULES.md) for the full enforcement checklist):

- [ ] Exactly **one** Topbar per page
- [ ] Left: **`Avatar`** application mark (**small**, **fjord**) → app name breadcrumb (link to home when object open, plain text otherwise) → object name breadcrumb (only when object is in focus)
- [ ] Breadcrumb segments are interactive links — not static text
- [ ] Object dropdown (if present) only on the object name segment; actions are object-scoped only
- [ ] Inline metadata (if present) is a plain string, **left-aligned after** the breadcrumb — not centered
- [ ] Middle: **Tabs** or **Segmented control** at **small** if present; no sidebar; no primary CTA in the Topbar
- [ ] **Primary / app-specific actions** live in the **content area below** the Topbar
- [ ] Right strip when used: **Share → Notifications → Theme → Atlas → user Avatar**; Share / Notifications / Theme use **ghost** **small** icon buttons; Atlas **secondary** **small** with leading icon + "Atlas"
- [ ] Theme: **sun** in light mode, **moon** in dark mode; **Menu** with **Light mode** / **Dark mode** and **checkmark** on the active row; `setTheme('light' | 'dark')` (or equivalent) wired to `document.documentElement`
- [ ] `tailwind.config` has `darkMode: 'class'`
- [ ] `add-navigation` (`@cognite/dune-industrial-components/navigation`) removed if previously present

---

## Additional resources

- Full Topbar architecture rules: [RULES.md](RULES.md)
- Aura Topbar Storybook: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-topbar--docs
- Aura Breadcrumb Storybook: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-breadcrumb--docs
- Aura Button Storybook: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-button--docs
- Aura colors / dark mode tokens: https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-colors--docs
