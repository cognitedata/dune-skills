# Topbar Rules — Full Reference (Aura)

Detailed architecture and usage rules for the Topbar across Dune and Fusion applications. Read this file when you need the full rule set beyond the quick reference in `SKILL.md`.

**Source:** `use-topbar.md` (Aura Topbar agent and developer rules)
**Storybook:** https://cognitedata.github.io/aura/storybook/?path=/docs/primitives-topbar--docs
**Install:** `pnpm dlx shadcn@latest add @aura/topbar`

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

1. **Application icon** — Aura-provided asset/component for the app.
2. **Breadcrumbs** — Aura Breadcrumb component.

Always show:

- Application icon on the far left.
- Application name in the **first** breadcrumb segment, or as the prefix when no object is open.
- Current object name in the **last** breadcrumb segment **only** when a specific object is open; otherwise omit.

Object name + optional menu:

- The object name may act as the **sole** trigger for an object-level dropdown when needed.
- Dropdown items are delivered via Aura/platform patterns; applications configure which items appear.
- Use Aura components for trigger, menu, and items only.

### 3.2 Middle section (optional)

- Used for **non-interactive** metadata (e.g. last updated, created by, read-only state).
- If unused, **leave the region empty**; do not collapse layout in a way that breaks left/right alignment.

### 3.3 Right section

Two parts, in order:

#### A. Flexible slot (app-controlled)

Supports, as needed:

- Dark mode toggle (always present — see `SKILL.md §4–5`)
- Buttons, avatars, tabs, app-level navigation
- Primary CTA: Aura **Default** button, **size small**, **rightmost** among app controls, **at most one**.

#### B. Persistent system actions (far right — platform-controlled)

Not end-user configurable. Secondary-style icon buttons, Aura icons, size small, fixed order:

1. Agent experience
2. Notifications
3. Share

Rules:
- Order is fixed; users cannot reorder.
- Visibility may be toggled per product policy; apps may hide individual actions.
- Styling and behavior must **not** be overridden by app teams.
- Spacing between flexible slot and this group is defined by Aura Topbar.

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
| Left: icon + breadcrumbs | App (via Aura API) | App name + optional object name per rules above |
| Object dropdown contents | App (items), platform/Aura (presentation) | Trigger = object name only |
| Middle metadata | App | Optional; non-interactive |
| Flexible right slot | App | Dark mode toggle (always) + primary CTA (one Default button, small, rightmost) |
| System actions (agent, notifications, share) | Platform + visibility rules | Fixed order; apps may hide; no style/behavior overrides |
| Theming | Aura tokens only | No arbitrary CSS |

**Open items (to be finalized):**

- System-level configuration matrix (tenant vs build-time vs runtime) per action.
- Telemetry/analytics for notifications, share, agent.
- Shell responsibility details (single mount point vs per-app composition).
- Lint rules and automated checks for Topbar compliance.
- Automated binding of app config / Fusion config to pre-fill agent answers.

---

## 9. Configuration interview protocol

Before implementing or changing Topbar wiring, run the structured interview in `SKILL.md §2`. Key rules:

- Ask one question at a time; wait for answer.
- Branch: skip questions that don't apply.
- Prefer existing config (Dune/Fusion manifest) over asking again; state what was inferred.
- Suggest best practices when user is unsure.
- Avoid over-configuration: do not add middle metadata, object dropdowns, or extra buttons unless user confirms.
- End with a recap of left / middle / right / system actions / omissions before implementing.

### Best-practice nudges

- **Breadcrumbs:** One segment for app name when no object; add object segment only when UI is scoped to that object.
- **Object menu:** Only add if users need quick object-scoped actions without leaving context.
- **Middle:** Omit unless metadata is always relevant and read-only.
- **Right slot:** Dark mode toggle always present. One Default primary CTA at most; everything small.
- **System actions:** Hide what the product does not need; never restyle them.

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
- Put application name and object name (when applicable) in breadcrumbs per left-section rules.
- Use **small** size for Topbar controls; one Default primary button max in the flexible slot.
- Include a dark mode toggle in the right slot (always).
- Leave middle empty when unused; rely on Aura for alignment.
- Respect fixed order and Aura styling for system icon buttons.

**Don't**

- Don't build a custom top bar or duplicate global chrome.
- Don't use multiple Topbars or a second header in embedded views.
- Don't override system action appearance or behavior.
- Don't add two Default/primary CTAs.
- Don't use non-token styling on Topbar or its children.
- Don't use `@cognite/dune-industrial-components/navigation` (deprecated — use `@aura/topbar`).

---

## 12. Enforcement

1. Verify `@aura/topbar` (or documented shell wrapper) is the only top-level app chrome for navigation branding.
2. Reject PRs or changes that introduce parallel Topbar implementations or extra top navigation bars.
3. Check left section order: icon → breadcrumbs with correct name/object rules.
4. Check right slot: dark mode toggle present; at most one Default button, small, rightmost among app controls.
5. Check system actions: correct order when multiple visible; no custom styling on those controls.
6. Confirm the configuration interview (§9 / `SKILL.md §2`) was completed for new Topbar work, or that changes are narrow enough that a partial re-run of affected questions is sufficient.

---

## 13. Revision history intent

Update `use-topbar.md` (source) and regenerate this file when:

- Aura Topbar API or package name changes.
- Platform finalizes configuration, telemetry, shell mount, or lint rules.
- New global system actions are added to the persistent group.
- Dune/Fusion config paths or fields used for pre-flight are standardized.
