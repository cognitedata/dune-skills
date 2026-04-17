# Topbar Rules — Full Reference (Aura)

Detailed architecture and usage rules for the Topbar across Dune and Fusion applications. Read this file when you need the full rule set beyond the quick reference in `SKILL.md`.

**Storybook:** https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-topbar--docs
**Install:** `pnpm dlx shadcn@latest add @aura/topbar` — registry component only; do not use `npm install` / `pnpm add` / `yarn add`

---

## 1. Non-negotiables

1. Every authenticated app view must render exactly **one** Topbar, composed only from Aura Topbar primitives and `@aura/topbar` documented APIs.
2. Do **not** implement a custom top bar, duplicate header, or alternate app chrome that replaces or shadows the Topbar.
3. Do **not** render **multiple** Topbars on a single page (including embedded views or nested frames).
4. Styling and behavior must follow Aura: **token-level theming only**. No ad-hoc overrides that break Aura semantics.

---

## 2. Where Topbar is omitted

The Topbar **may** be omitted only on:

- Login / auth-only screens
- Fullscreen modal or fullscreen flows that hide global chrome by design
- Other explicit shell exceptions documented by the platform team

If unsure whether a route qualifies, **default to including the Topbar**.

---

## 3. Layout contract (three regions)

### 3.1 Left section (required)

Order (left → right):

1. **Application icon** — filled icon from Aura's connected Tabler library. Priority: (1) custom icon from app config, (2) relevant filled Tabler icon, (3) `IconAppsFilled` as the fallback. Never use an outlined icon variant.
2. **Breadcrumbs** — Aura Breadcrumb component. All segments are **interactive links** that navigate to their corresponding route. Do not render breadcrumbs as static/non-interactive text.
3. **Inline metadata** _(optional)_ — a plain string immediately to the right of the breadcrumb (e.g. "Updated 3 hours ago", "Read-only"). String only; no links, icons, or interactive elements. Omit entirely when unused.

Breadcrumb rules:

- Application name is always the **first** segment.
- When an object is open, clicking the app name navigates back to the app's home/root route.
- When no object is open, the app name is **not** a link (it is the current location).
- Current object name appears in the **last** segment **only** when a specific object is open; otherwise omit.

Object dropdown:

- The object name may act as the **sole** trigger for an object-level dropdown **only when an object is open**.
- **No dropdown on the app name.** If app-level settings are needed, add a button in the action slot.
- All dropdown actions must apply **only to the currently open object** (rename, duplicate, export, delete, etc.). Do not mix in app-level or global actions.
- Use Aura components for the trigger, menu, and items only.

### 3.2 Middle section — navigation (optional)

- Used for **global navigation tabs** — either between pages/routes (multi-page app) or between views/panels within a single-page app.
- Tabs are the **preferred** navigation pattern. If the app has distinct views, use tabs here.
- **Never use a sidebar** for primary app navigation. If additional sub-navigation is needed beyond these tabs, it must live within the content area.
- If the app has no global navigation or only one view, leave this section empty entirely.

### 3.3 Right section

Two parts, in order:

#### A. Action slot (app-controlled)

- App-wide or every-page action buttons only.
- **The gate test:** If an action does not apply to the entire app or every page, it does not belong here — place it in the content area below the Topbar.
- Primary CTA: Aura **Default** button, **size small**, **rightmost**, **at most one**. All other buttons are secondary.

#### B. Persistent system actions (far right — platform-controlled)

Not configurable by app teams. Secondary-style icon buttons, Aura icons, size small, fixed order:

1. Agent experience _(optional)_
2. Notifications _(optional)_
3. Share _(optional)_
4. **Dark mode toggle** _(always present)_ — moon icon in light mode, sun icon in dark mode
5. **User avatar** _(always present)_

Rules:
- Order is fixed; apps cannot reorder.
- Agent, Notifications, and Share visibility may be toggled per product policy; apps may hide them individually.
- Dark mode toggle and avatar are **always visible** — they must not be hidden.
- Styling and behavior must **not** be overridden by app teams.
- Spacing between action slot and this group is defined by Aura Topbar.

---

## 4. Sizing

All Topbar interactive elements use **small** size unless Aura Topbar documentation explicitly prescribes otherwise.

---

## 5. Responsive behavior

- Follow Aura Topbar default responsive behavior.
- If an app places many items in the flexible slot, the app must handle overflow (prioritization, fewer items, progressive disclosure). This is an **app responsibility** when content is dense.

---

## 6. Accessibility & keyboard

- Tab / focus order follows **visual** order: left section → middle (if any) → flexible right slot → persistent system actions.
- No extra skip link or focus management requirements beyond Aura defaults at this time.

---

## 7. Loading & long labels

- Truncation, ellipsis, tooltips, loading states: follow Aura default behavior for Topbar and related primitives. Do not invent one-off patterns.

---

## 8. Configuration model

| Concern | Who controls | Notes |
|---------|-------------|-------|
| Topbar presence & single instance | App + shell | One per page; no duplicates |
| Left: icon | App (via Aura API) | Filled Tabler icon; priority: config → relevant filled → `IconAppsFilled` |
| Left: breadcrumbs | App (via Aura API) | Interactive links; app name navigates to home only when object is open |
| Left: inline metadata | App | Optional plain string after breadcrumb; no interactive elements |
| Object dropdown | App (items), platform/Aura (presentation) | Object name only, object-scoped actions only; no app-level dropdown |
| Middle: navigation tabs | App | Optional; preferred for multi-page nav and SPA view switching; never a sidebar |
| Right: action slot | App | App-wide or every-page actions only; one Default CTA max, small, rightmost |
| System actions (agent, notifications, share, dark mode, avatar) | Platform + visibility rules | Fixed order: agent → notifications → share → dark mode → avatar; dark mode and avatar always visible; no style/behavior overrides |
| Theming | Aura tokens only | No arbitrary CSS |

**Open items (to be finalized):**

- System-level configuration matrix (tenant vs build-time vs runtime) per action.
- Telemetry/analytics for notifications, share, agent.
- Shell responsibility details (single mount point vs per-app composition).
- Lint rules and automated checks for Topbar compliance.
- Automated binding of app config / Fusion config to pre-fill agent answers.

---

## 9. Configuration interview protocol

The full interview is defined in `SKILL.md §2` (Step 2, pre-flight through closing summary). Run it before implementing or changing any Topbar wiring.

---

## 10. Composition & API guidance

- Prefer the Aura Topbar API as documented in Storybook. Use documented props and slots for each region.
- Structured data (names, breadcrumb items, menu definitions) is easier to validate than arbitrary JSX.
- Where Aura exposes slots, only place Aura-approved components inside each slot.
- Do **not** bypass the Topbar by injecting a second header row or fake breadcrumbs outside the Topbar.
- Always fetch actual component names, props, and slot names from the current Aura package and Storybook before writing code — the pseudo-code in this document is illustrative only.

---

## 11. Do / Don't

**Do**

- Use `@aura/topbar` and compose Topbar exactly as Aura documents.
- Keep **one** Topbar per page.
- Use a filled Tabler icon; fall back to `IconAppsFilled` if nothing relevant exists.
- Make breadcrumb segments interactive links that navigate to their routes.
- Add the app name link to home/root only when an object is currently open.
- Put object dropdown only on the object name; scope all actions to that object only.
- Use the middle section for navigation tabs (multi-page routes or SPA view switching).
- Put action buttons in the right slot only if they apply to the entire app or every page.
- Use **small** size for all Topbar controls; one Default primary button max in the action slot.
- Always include the dark mode toggle in system actions — moon icon in light mode, sun icon in dark mode.
- Always include the user avatar in system actions.
- Respect fixed order and Aura styling for system icon buttons.

**Don't**

- Don't build a custom top bar or duplicate global chrome.
- Don't use multiple Topbars or a second header in embedded views.
- Don't use a sidebar for navigation — ever. Use middle tabs or content-area navigation instead.
- Don't put page-specific action buttons in the topbar — they belong in the content area.
- Don't add a dropdown to the app name — use a button in the action slot for app settings.
- Don't mix app-level and object-level actions in the object dropdown.
- Don't render breadcrumbs as static/non-interactive text.
- Don't use an outlined icon — always use the filled variant.
- Don't override system action appearance or behavior.
- Don't add two Default/primary CTAs.
- Don't place the dark mode toggle in the action slot — it belongs in system actions.
- Don't hide or omit the dark mode toggle or user avatar.
- Don't use non-token styling on Topbar or its children.
- Don't use `@cognite/dune-industrial-components/navigation` (deprecated — use `@aura/topbar`).

---

## 12. Enforcement

1. Verify `@aura/topbar` is the only top-level app chrome; reject any parallel header implementation.
2. Check left section: filled icon → breadcrumbs (interactive links) → optional inline metadata (string only).
3. Check breadcrumb behavior: app name links to home only when object is open; object dropdown (if any) is on the object name only and contains only object-scoped actions.
4. Check middle section: navigation tabs if present; confirm no sidebar exists anywhere in the app.
5. Check action slot: every button applies to the entire app or every page; no page-specific actions; at most one Default CTA, small, rightmost.
6. Check system actions: correct fixed order; dark mode toggle and avatar always present; moon in light mode, sun in dark mode; no custom styling on any system action.
7. Confirm the configuration interview (`SKILL.md §2`) was completed for new Topbar work.

---

## 13. Revision history intent

Update `RULES.md` when:

- Aura Topbar API or package name changes.
- Platform finalizes configuration, telemetry, shell mount, or lint rules.
- New global system actions are added to the persistent group.
- Dune/Fusion config paths or fields used for pre-flight are standardized.
