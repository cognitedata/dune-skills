---
name: use-topbar
description: >-
  Wires the Aura Topbar (@aura/topbar) into a Dune or Fusion app as the single
  top navigation bar with breadcrumbs, optional middle metadata, right-slot
  actions, and a built-in dark mode toggle. Use when adding a topbar, top
  navigation bar, app chrome, navigation header, breadcrumbs, or dark mode to a
  Dune app. Also use when replacing or deprecating the add-navigation skill, or
  when the user mentions topbar, Topbar, top nav, app header, or dark mode
  toggle in a Dune or Fusion app context. Also use when the user is starting,
  building, creating, or setting up a new Dune application, Fusion app, or
  Dune-based project — including any mention of "new Dune app", "Dune
  application", "building a Dune app", "starting a Fusion app", or any request
  to scaffold, wire up, or begin work on a Dune or Fusion application.
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
- Any Dune/Fusion app config (`app.config.ts`, `fusion.config.ts`, manifest files) — `displayName`, `name`, `icon` fields

Note what you find. If app name, icon, or a dark mode hook already exists, apply those as defaults and skip the corresponding interview questions. State what was inferred.

---

## Step 2 — Configuration interview (mandatory)

**Complete this interview before writing any implementation code.** Do not skip, shorten, or defer it. If completing it mid-task feels disruptive, pause the task, run the interview, then resume.

Ask **one question at a time** and wait for the answer. Skip only questions that Step 1 already answered definitively.

### Topbar layout reference

Use this diagram to orient yourself and the user throughout the interview:

```
┌────────────────────────────────────────────────────────────────────────────────────────────────────┐
│  [Icon]  [App Name]  >  [Object ▾]  [metadata]  │  [Tab 1]  [Tab 2]  [Tab 3]  │  [+ Action]  │ ⚙🔔🔗☀/🌙👤 │
│  ←─── Left: icon + breadcrumb + metadata ──────  │  ←── Middle: navigation ──→ │  ←─ Actions  │  System   │
└────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

**Left section — breadcrumb states:**

```
No object open:        [Icon]  My App
Object open:           [Icon]  My App  >  Root Cause Analysis ▾
With inline metadata:  [Icon]  My App  >  Root Cause Analysis ▾   Updated 3 hours ago
```

- App name clicking navigates to the app's home/root route — but only when an object is open. If no object is open, the app name is not a link.
- Object name is clickable only when it acts as a dropdown trigger (▾). Each breadcrumb segment is an interactive link that navigates to its route.

**Middle section — navigation tabs (optional):**

```
[Overview]  [Well analysis]  [Pump analysis]
```

- Used for multi-page global navigation OR view switching within a single-page app.
- Omit entirely if the app has no navigation or only one view.

**Right section breakdown:**

```
Action slot (app-controlled)          System actions (fixed order, far right)
──────────────────────────────────    ────────────────────────────────────────
[Export]  [+ Add data]                [Atlas]  [🔔]  [Share]  [☀/🌙]  [👤]
 ← app-wide actions only →  CTA        optional  opt   opt    ALWAYS   always
                                        never reordered or restyled
```

**Theme toggle icon logic:**
- Light mode → show **moon icon**
- Dark mode → show **sun icon**

---

### Left section

**Q1 — App icon**

> "Looking at the left section of the diagram — every app has a small icon at the far left. Does your app already have an icon defined, or should I suggest one?"

- **Priority order:**
  1. Custom icon already defined in Dune/Fusion app config → apply it and skip.
  2. No custom icon → suggest a relevant filled Tabler icon based on the app's name/purpose (e.g. monitoring → `IconActivityHeartbeat`, analytics → `IconChartBar`). Confirm the suggestion before proceeding.
  3. Nothing obvious fits → use `IconAppsFilled` as the fallback.
- Always use a **filled** icon variant from Aura's connected Tabler library. Never use outlined icons.

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
- **There is no dropdown on the app name.** If users need app-level settings (e.g. manage permissions, configure defaults), add a button in the action slot instead.
- All actions in the object dropdown must apply **only to the currently open object** — do not mix in app-level or global actions.
- Examples: "Rename this canvas", "Duplicate this report", "Export this document", "Delete this item".
- If yes: "What object-specific actions should appear in the menu?"

---

### Left section — inline metadata

**Q5 — Inline metadata** _(optional)_

> "Would you like to show a short status string directly after the breadcrumb — things like 'Updated 3 hours ago', 'Read-only', or a badge? This appears inline in the left section, right next to the object or app name."

- If yes: "What text should appear there?"
- **String only** — no links, icons, or interactive elements.
- Omit entirely if unused — do not add a placeholder.
- Typical use: last-modified time, read-only state, a status label tied to the current object or page.

---

### Middle section — navigation

**Q6 — Navigation tabs** _(optional)_

> "Looking at the middle section of the diagram — does your app have multiple pages or views users switch between? This is where global navigation tabs live."

- **Multi-page app:** tabs navigate between routes (e.g. Overview → `/overview`, Settings → `/settings`).
- **Single-page app:** tabs switch between views or panels within the same page (e.g. Overview, Well analysis, Pump analysis).
- Tabs are the **preferred** navigation pattern for both cases. If the app has distinct views or sections, tabs here are strongly recommended.
- **Never use a sidebar for primary navigation.** If the app needs additional internal navigation beyond these tabs, it must live within the content area — not as a sidebar.
- Only include tabs that are relevant from every page (or always visible in the app). Page-specific sub-navigation belongs in the content area.
- If yes: "What are the tab names, and where does each one navigate?"

---

### Right section — actions

**Q7 — Global action buttons** _(optional)_

> "Are there any action buttons that should always be visible in the top bar — like '+ Add data', 'Export', or 'Create new [item]'?"

**Before adding any button, apply this test:** Does this action apply to the **entire app** or to **every page** in the app? If it only makes sense on one specific page or in one context, it must stay in the **content area below the topbar** — not here.

- If yes: "What are the button labels and what does each one do?"
- At most **one** primary CTA (`variant="default"`, `size="sm"`, rightmost). All others are secondary.
- When in doubt, leave the action slot empty and place the button on the page where it's relevant.

**Q8 — System actions** _(ask each sub-question separately)_

> "The fixed system actions on the far right are platform-controlled and always appear in the same order. The dark mode toggle and avatar are always present. Which optional ones does your app need?"

Ask each separately:

- **Atlas:** "Does this app use Atlas, Cognite's AI assistant? Should the Atlas button appear?"
- **Notifications:** "Does this app send alerts or notifications? Should the Notifications button appear?"
- **Share:** "Do users need to share content with others? Should the Share button appear?"

The dark mode toggle and user avatar are **always included** — do not ask about them.

Fixed order (left → right): **Atlas → Notifications → Share → Dark mode toggle → Avatar**

Apps may hide Atlas, Notifications, and Share individually, but may never reorder or restyle any system action.

---

### Omissions

**Q9 — Excluded routes**

> "Are there any screens where the top bar should NOT appear? Common exceptions: login/auth screens, fullscreen flows, onboarding. Default is to show it everywhere."

---

**Closing:** Before implementing, summarize the configuration in five bullets or fewer: left (icon + breadcrumb pattern + inline metadata if any), middle (navigation tabs or none), right action slot (which buttons, if any), system actions (Atlas/Notifications/Share on/off — dark mode and avatar always on), excluded routes. Then proceed to Step 3.

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

Always implement dark mode. Check for an existing theme system first:

- Search for `useDarkMode`, `useTheme`, `useColorScheme`, or a `ThemeProvider` in `src/`
- If found, wire into it and skip creating a new hook.

If none exists, create `src/hooks/use-dark-mode.ts`:

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

Apply the initial class on page load in `main.tsx` / `index.tsx`:

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
import { useDarkMode } from '@/hooks/use-dark-mode';

export function AppShell({ children }: { children: React.ReactNode }) {
  const { isDark, toggle } = useDarkMode();

  return (
    <>
      <Topbar
        // Left — required
        // Use a filled Tabler icon. Priority: app config icon → relevant filled icon → IconAppsFilled.
        applicationIcon={<AuraAppIcon name="..." />}
        breadcrumbs={
          <Breadcrumb>
            {/* App name: link to home/root only when an object is open; plain text otherwise */}
            <BreadcrumbItem label="Application Name" href="/" />
            {/* Object segment: add only when a domain object is open; omit otherwise */}
            {/* <BreadcrumbItem label={objectName} onDropdownOpen={...} /> */}
          </Breadcrumb>
        }

        // Inline metadata — optional string immediately after breadcrumb; omit when unused
        // breadcrumbMetadata="Updated 3 hours ago"

        // Middle — optional navigation tabs; omit when app has no global navigation
        // Use for multi-page route navigation OR view switching within a single-page app.
        // Never use a sidebar — all primary navigation lives here or in the content area.
        centerSlot={
          null
          // Example: <Tabs value={currentTab} onValueChange={navigate}>
          //   <Tab value="overview" label="Overview" />
          //   <Tab value="details" label="Details" />
          // </Tabs>
        }

        // Right action slot — app-wide or every-page actions only
        // If an action only applies to one page, it belongs in the content area, not here.
        trailingSlot={
          <>
            {/* Secondary action buttons here */}
            {/* Primary CTA — at most one, rightmost, variant="default", size="sm" */}
            {/* <Button variant="default" size="sm">+ Add data</Button> */}
          </>
        }

        // System actions — fixed order: agent → notifications → share → darkMode → avatar
        // Dark mode and avatar are always present. Agent/notifications/share are optional.
        // Verify exact prop names and API shape against @aura/topbar Storybook.
        systemActions={{
          agent: { visible: true },
          notifications: { visible: true },
          share: { visible: true },
          darkMode: {
            visible: true,   // always true — never hide
            isDark,
            onToggle: toggle,
            // Light mode → moon icon; dark mode → sun icon
          },
          avatar: { visible: true },
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
    {/* page content */}
  </AppShell>
</div>
```

---

## Step 6 — Compliance checklist

Verify before finishing (see [RULES.md §12](RULES.md) for the full enforcement checklist):

- [ ] Exactly **one** Topbar per page
- [ ] Left: filled icon → app name breadcrumb (link to home when object open, plain text otherwise) → object name breadcrumb (only when object is in focus)
- [ ] Breadcrumb segments are interactive links — not static text
- [ ] Object dropdown (if present) only on the object name segment; actions are object-scoped only
- [ ] Inline metadata (if present) is a plain string — no interactive elements
- [ ] Middle: navigation tabs (if present) handle all global navigation; no sidebar used anywhere
- [ ] Action slot: every button applies to the entire app or every page; page-specific actions are in the content area
- [ ] At most **one** `variant="default"` button in the action slot, `size="sm"`, rightmost
- [ ] System actions in fixed order (agent → notifications → share → dark mode → avatar); no style overrides
- [ ] Dark mode toggle always visible; moon icon in light mode, sun icon in dark mode
- [ ] `tailwind.config` has `darkMode: 'class'`
- [ ] `add-navigation` (`@cognite/dune-industrial-components/navigation`) removed if previously present

---

## Additional resources

- Full Topbar architecture rules: [RULES.md](RULES.md)
- Aura Topbar Storybook: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-topbar--docs
- Aura Breadcrumb Storybook: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-breadcrumb--docs
- Aura Button Storybook: https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-button--docs
- Aura colors / dark mode tokens: https://cognitedata.github.io/aura/storybook/?path=/docs/foundations-colors--docs
