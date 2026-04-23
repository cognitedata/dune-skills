# Writing copy

## Role

You are writing interface copy for Cognite customer-facing applications. Every piece of UX text must be purposeful, concise, conversational, and clear. Always identify the target audience persona before writing — the persona determines reading level, technical vocabulary, and tone.

For code-level accessibility (keyboard navigation, ARIA, focus, headings, live regions), see `handling-states.md`.

## Audience personas

Canonical persona definitions live in the cogdocs repository (`cogdocs/cogdocs-metadata.mdx`, **Audience** section). This summary covers what matters for microcopy decisions.

| Persona | Technical level | UX copy implication |
|---|---|---|
| `businessUser` | Low | Plain language; outcomes over features; domain terms OK, avoid platform jargon |
| `businessDecisionMaker` | Low | Plain language; ROI, business value, strategic impact; minimal technical detail |
| `appMaker` | Mid | Configuration, automation, outcomes; avoid deep code/API detail |
| `dataAnalyst` | Mid | Analytics, insights, dashboards; data terms OK, keep explanations clear |
| `partner` | Mid–high | Precise; balance technical accuracy with clarity |
| `administrator` | High | Technical terms OK; reliability, security, compliance, access; be precise |
| `dataEngineer` | High | Technical terms OK; pipelines, ingestion, transformation |
| `developer` | High | Technical terms OK; APIs, SDKs, integrations; precise and concise |
| `aiEngineer` | High | Technical terms OK; ML/AI, models, automation |
| `dataScientist` | High | Technical terms OK; experiments, models, analytics |
| `securityEngineer` | High | Technical terms OK; IAM, threats, compliance |
| `solutionArchitect` | High | Technical terms OK; integration, strategy, best practices |
| `internal` | Varies | Can use Cognite-internal jargon; match internal conventions |

**Reading level:** Low = 7th–8th grade; Mid = 9th–10th grade; High = 10th–11th grade.

When the persona is unknown, default to plain language and outcomes.

## Voice and tone

Voice is consistent; tone adapts to the user's emotional state.

| Scenario | Tone | Example |
|---|---|---|
| First-time onboarding | Friendly, welcoming | "Let's get started — Cognite Data Fusion is ready when you are." |
| Technical documentation | Clear, direct, supportive | "Configure your endpoint and authenticate using your API key." |
| Error messages | Empathetic, constructive | "Something went wrong. Try refreshing, or check your connection." |
| Success states | Encouraging, concise | "Your data is now flowing." |
| Product tours / help | Conversational, helpful | "Want a quick tour? We'll walk you through the essentials in under 2 minutes." |
| High-stakes actions | Serious, transparent | "Delete pipeline? All history will be permanently removed." |

## Grammar and style

### Language and capitalization

- **American English**: color, center, organization, modeling
- **Sentence case everywhere**: "Create data model" — not "Create Data Model". No exceptions for UI text. Only proper nouns and product names are capitalized: Cognite Data Fusion, OPC-UA, Aura.
- **No all-caps**
- **No "CDF" in UI copy** — customers may white-label the platform; use product or feature names instead

### Numbers and units

- **Numerals for all numbers**, including those under 10: "6 queries", "3 items", "1 result"
- Non-breaking space between number and unit: "50 Mbps"
- Don't use "(s)" or "(es)" — choose singular or plural based on context

### Abbreviations and punctuation

- No Latin abbreviations: use "for example" not "e.g.", "and more" not "etc."
- Define acronyms and technical terms when first used (unless writing for technical personas)
- No ampersands (&): use "and" — including in headings
- **Oxford comma**: "apples, oranges, and pears"
- No exclamation marks in UI copy
- No period after labels, tooltip text, or single-sentence bulleted list items; use periods for multiple/complex sentences
- Ellipsis (…): only for ongoing processes or truncated text — use sparingly

### Pronouns

- Don't mix "my" and "your" in the same context
- **"My [resource]"** for app-owned items: "My data", "My assets"
- Minimize "I" and "we" representing the application; focus on the user's perspective
- Avoid ambiguous pronouns ("this", "that") without an explicit referent — name the thing

## Action labels

Use sentence case with an object: "Edit model", "Delete asset".

### Approved labels

| Label | Use when |
|---|---|
| Add | Taking an existing object into a new context ("Add to canvas") |
| Apply | Setting filtered values that affect subsequent system behavior |
| Approve | User agrees; initiates next step in a business process |
| Back | Returning to the previous step in a sequence or hierarchy |
| Cancel | Stopping the current action or closing a modal — warn of data loss |
| Clear | Clearing all fields/selections; restores defaults |
| Close | Closing a page, panel, or secondary window — often icon-only |
| Copy | Copying an object to the clipboard |
| Create | Making a new object from scratch |
| Delete | Permanently destroying an object |
| Discard | Discarding unsaved changes during create/edit |
| Download | Transferring a file from remote to local |
| Duplicate | Creating a copy in the same location as the original |
| Edit | Changing data/values of an existing object |
| Export | Saving data in an external format; typically opens a dialog |
| Import | Bringing data from an external source; typically opens a dialog |
| Next | Advancing to the next step in a wizard |
| Finish | Completing a multi-step wizard |
| Open | Opening a drawer, modal, or new page within current context |
| Publish | Making content available to intended users |
| Refresh | Reloading a view that is out of sync with the source |
| Register | Creating a new user account |
| Remove | Removing an object from the current context without destroying it |
| Reset | Reverting to last saved or default state |
| Save | Saving pending changes without closing the window/panel |
| Search | Goal-oriented action to find precise information |
| Select | Choosing one or more options from a list |
| Show / Hide | Revealing or removing an element from view without deleting — use as a pair |
| Sign in / Sign out | Entering or exiting the application |
| Undo / Redo | Reversing or re-applying the most recent action |
| Upload | Transferring a file from local to remote |
| View | Presenting additional information or properties for an object |

### Labels to avoid

| Avoid | Use instead | Reason |
|---|---|---|
| Confirm | The specific action verb ("Delete", "Send") | Too vague |
| Log in / Log out | Sign in / Sign out | "Log" is technical jargon |
| Sign up | Register | Avoids confusion with "Sign in" |
| Submit, OK, Yes | The specific outcome verb | Generic; tell users what happens |
| Click here, Read more | Descriptive link text | Inaccessible; not input-agnostic |

## UI text patterns

### Titles
Noun phrases, sentence case. Examples: "Asset overview", "Pipeline runs", "Configure integration"

### Buttons and CTAs
Active imperative verb + object. 2–4 words target, 6 max. Examples: "Save changes", "Delete pipeline", "View details"

### Error messages
Pattern: `[What failed]. [Why/context if known]. [What to do].`
Examples:
- "Ingestion failed. Check your extractor configuration and try again."
- "Couldn't save changes. Connection lost. Reconnect and retry."
Avoid: blame language, dead ends with no recovery path

### Success messages
Past tense, specific, brief. Pattern: `[Action] [result]`
Avoid "successfully"; that's implied in the pattern
Examples: "Changes saved", "Pipeline started", "Integration configured"

### Empty states
Explanation + CTA. Example: "No assets yet. Connect a data source to start exploring."

### Tooltips
One to two sentences, present tense. Pattern: `[What it is]. [What it does or why it matters].`
Examples:
- "Asset ID. The unique identifier for this asset in Cognite Data Fusion."
- "Time granularity. Controls how data points are aggregated in the chart."
Never repeat the label. Never write more than 2 sentences.

### Confirmation dialogs
State the consequence, not just the action. Pattern: `[What will be lost or affected]. [Reversibility]. [Specific action].`
- Primary CTA: match the specific action ("Delete pipeline", not "Confirm")
- Secondary CTA: always provide a clear exit ("Cancel")
Examples:
- "Delete pipeline? All runs and history will be permanently removed. This can't be undone."
- "Remove team member? They'll lose access to all shared resources immediately."
Avoid: "Are you sure?", manipulative phrasing

### Form fields
- **Labels**: Clear noun phrases ("Time series ID", "Email address")
- **Placeholder text**: Use sparingly, only for standard formats like "name@example.com"
- **Helper text**: Verb-first; explain why the information is needed

### Notifications
Verb-first title + contextual description. 10–15 words total.
Example: "Extractor disconnected. Check your network and reconnect."

## Accessibility

- Use **"Select"** not "Click" — input-agnostic: mouse, keyboard, touch, voice
- Avoid ambiguous pronouns — screen readers lose surrounding context
- Write descriptive link text: "Read pricing details" not "Click here"
- Alt text by image type:
  - Icon → describes function: "Download PDF" not "download icon"
  - Link image → describes destination: "Contact support" not "question mark"
  - Chart/diagram → summarizes meaning: "Bar chart showing pipeline throughput declining 20% in Q3"
  - Decorative image → empty alt text (`alt=""`)
  - Never write "image of" or "photo of"
- For charts and metrics, describe key trends or values in adjacent text — don't rely on visual encoding alone
- Target 8–14 words per sentence (8 = 100% comprehension, 14 = 90%)
- Pair visual indicators with text: "Error: field required" alongside a red icon

## Date and time formatting

- **Prefer written dates**: "2 January 2023" not "02/01/2023"
- **Relative vs absolute**: ≤24 h from now → relative ("32 min ago"); >24 h → absolute ("2 Jan 2023")
- Always include the year unless obvious from context
- No ordinal numbers: "2 January" not "2nd January"
- Separate date and time with "at": "2 Jan 2023 at 10:00 AM" — no comma
- **12-hour time**: uppercase AM/PM, no periods, space before: "10:00 AM"
- **Time zone**: UTC only; spell out "UTC" in text-only contexts
- Never make the user convert time zones — handle in code
- Ranges: consistent format across start and end; for ongoing processes use absolute start + "ongoing" until complete
- Duration: no comma between units ("10 minutes 3 seconds"); space between number and unit in running text ("3 min"); no space in controls ("3min")

**Time unit abbreviations** (no periods; same form singular/plural):
ms, s, min, hr, d, wk, mo, yr

**Day abbreviations** (3 chars for i18n):
Mon, Tue, Wed, Thu, Fri, Sat, Sun

**Month abbreviations** (4 chars for i18n):
Jan, Feb, Mar, Apr, May, Jun, Jul, Aug, Sep, Oct, Nov, Dec

## Localization

- Keep sentences short with the subject near the start — compound clauses increase translation cost
- Maintain consistent terminology and capitalization across strings (critical for translation memory)
- No Latin abbreviations in translatable strings: "for example" not "e.g.", "and more" not "etc."
- Avoid idioms and cultural references
- No ampersands: use "and"
- Small words (a, the, that, is): include in prose; may omit only in space-constrained labels and CTAs

## Benchmarks

| Element | Target | Maximum |
|---|---|---|
| Buttons / CTAs | 2–4 words | 6 words |
| Titles | 3–6 words, 40 characters | — |
| Tooltips | 10–20 words | 2 sentences |
| Error messages | 12–18 words | — |
| Instructions | 14 words | 20 words |
| Notifications | 10–15 words total | — |
| Line length | 40–60 characters | 70 characters |
