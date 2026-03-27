---
name: design-content-guidelines
description: Creates clear, user-centered interface copy following UX writing best practices. Use when writing buttons, labels, errors, empty states, headings, form fields, notifications, confirmations, tooltips, or any user-facing text. Includes voice/tone framework, accessibility guidelines, and research-backed benchmarks.
allowed-tools: Read, Glob, Grep, Edit, Write
---

## Role
You are writing interface copy for Cognite customer-facing applications.
Every piece of UX text must be purposeful, concise, conversational, and clear.
Always identify the target audience persona before writing — the persona
determines reading level, technical vocabulary, and tone.
See `references/cognite-audience.md` for all Cognite personas.

For code-level accessibility (keyboard nav, ARIA, focus), see **design-accessibility** skill.


<when-to-reference>
Consult this skill whenever you are:
- Writing or editing any user-facing text
- Creating buttons, labels, titles, messages, or form fields
- Writing error messages, success messages, or empty states
- Designing notifications, tooltips, or confirmation dialogs
- Designing onboarding experiences
- Auditing content for consistency and usability
</when-to-reference>

<file-routing>
Load these files based on the task:
- **Always**: `references/cognite-audience.md` — identify target persona before writing
- **Error messages**: `templates/error-message-template.md`
- **Empty states**: `templates/empty-state-template.md`
- **Onboarding flows**: `templates/onboarding-flow-template.md`
- **Translation, high-stress copy, or testing readability**: `references/accessibility-guidelines.md`
</file-routing>

<formatting-rules>
CASING — The single most common mistake in Cognite UI copy.

ALL interface text uses sentence case. No exceptions.

✅ "Save your changes"        ❌ "Save Your Changes"
✅ "Create new pipeline"      ❌ "Create New Pipeline"
✅ "Asset overview"           ❌ "Asset Overview"
✅ "Delete pipeline?"         ❌ "Delete Pipeline?"
✅ "No assets yet"            ❌ "No Assets Yet"

Only proper nouns and product names are capitalised:
Cognite Data Fusion, CDF, OPC-UA, Aura — not "pipeline", "asset", "dashboard", "integration"
</formatting-rules>


## Cognite Audience

Always check the target persona first. Persona determines reading level and vocabulary.

| Technical level | Personas | Writing approach |
|---|---|---|
| Low | `businessUser`, `businessDecisionMaker` | Plain language, lead with outcomes and benefits, avoid platform jargon |
| Mid | `appMaker`, `dataAnalyst`, `partner` | Domain terms OK, keep explanations clear |
| High | `developer`, `dataEngineer`, `administrator`, `aiEngineer`, `dataScientist` | Technical terms OK, be precise and concise |

When the persona is unknown, default to plain language and outcomes.

See `references/cognite-audience.md` for full persona descriptions.


## Core UX Writing Principles

Every piece of UX text should meet all four standards:

1. **Purposeful** — Helps users or the business achieve goals
2. **Concise** — Uses the fewest words possible without losing meaning
3. **Conversational** — Sounds natural and human, not robotic
4. **Clear** — Unambiguous, accurate, and easy to understand

### Key Best Practices

**Conciseness**
- Use 40-60 characters per line maximum
- Every word must have a job
- Break dense text into scannable chunks
- Front-load important information

**Clarity**
- Use plain language (7th grade reading level for general, 10th for professional)
- Avoid jargon, idioms, and technical terms unless the persona expects them
- Use consistent terminology throughout
- Choose meaningful, specific verbs

**Conversational Tone**
- Write how you speak
- Use active voice 85% of the time
- Include prepositions and articles
- Avoid robotic phrasing

**User-Centered**
- Focus on user benefits, not features
- Anticipate and answer user questions
- Use second-person ("you") language
- Match user's language and mental models


## UX Text Patterns

### Titles
- **Purpose**: Orient users to where they are
- **Format**: Noun phrases, sentence case
- **Types**: Brand titles, content titles, category titles, task titles
- **Examples**: "Asset overview", "Pipeline runs", "Configure integration"

### Buttons and Links
- **Purpose**: Enable users to take action
- **Format**: Active imperative verbs, sentence case
- **Pattern**: `[Verb] [object]`
- **Examples**: "Save changes", "Delete pipeline", "View details"
- **Avoid**: Generic labels like "OK", "Submit", "Click here"

### Error Messages
- **Purpose**: Explain the problem and provide a recovery path
- **Pattern**: `[What failed]. [Why/context if known]. [What to do].`
- **Examples**:
  - "Ingestion failed. Check your extractor configuration and try again."
  - "Couldn't save changes. Connection lost. Reconnect and retry."
- **Avoid**: Blame language, technical codes without explanation, dead ends
- For detailed types and templates → `templates/error-message-template.md`

### Success Messages
- **Purpose**: Confirm action completion
- **Format**: Past tense, specific, encouraging
- **Pattern**: `[Action] [result/benefit]`
- **Examples**: "Changes saved", "Pipeline started", "Integration configured"

### Empty States
- **Purpose**: Guide users when content is absent
- **Types**: First-use, user-cleared, no results
- **Format**: Explanation + CTA to populate
- **Example**: "No assets yet. Connect a data source to start exploring."
- For detailed types and templates → `templates/empty-state-template.md`

### Form Fields
- **Labels**: Clear noun phrases describing the input ("Time series ID", "Email address")
- **Instructions**: Verb-first, explain why the information is needed
- **Placeholder**: Use sparingly, only for standard formats like "name@example.com"
- **Helper text**: Static, on-demand, or automatic based on importance

### Notifications
- **Purpose**: Deliver timely, actionable information
- **Types**: Action-required (intrusive), Passive (less intrusive)
- **Format**: Verb-first title + contextual description
- **Example**: "Extractor disconnected. Check your network and reconnect."

### Tooltips
- **Purpose**: Explain a concept, field, or setting in context
- **Format**: One to two sentences, present tense
- **Pattern**: `[What it is]. [What it does or why it matters].`
- **Examples**:
  - "Asset ID. The unique identifier for this asset in Cognite Data Fusion."
  - "Time granularity. Controls how data points are aggregated in the chart."
- **Avoid**: Repeating the label, over-explaining, writing more than 2 sentences

### Confirmation Dialogs
- **Purpose**: Prevent accidental irreversible actions
- **Format**: State the consequence, not just the action
- **Pattern**: `[What will be lost or affected]. [Reversibility]. [Specific action].`
- **Examples**:
  - "Delete pipeline? All runs and history will be permanently removed. This can't be undone."
  - "Remove team member? They'll lose access to all shared resources immediately."
- **Primary CTA**: Match the action specifically ("Delete pipeline", not "Confirm" or "Yes")
- **Secondary CTA**: Always provide a clear exit ("Cancel")
- **Avoid**: Vague openers like "Are you sure?", manipulative phrasing


## Voice and Tone

### Voice (Consistent Brand Personality)
Voice is the consistent personality of the product. Define it using 3-5 concepts:

**Voice chart structure** — use this to document a product's voice:
```
### Concept: [Brand Principle]
Voice characteristics: [Adjective 1], [Adjective 2], [Adjective 3]
Do: [Example of text that embodies this]
Don't: [Example of what to avoid]
```

### Tone (Adaptive to Context)
Tone adapts to the user's emotional state while voice stays constant.

| Emotional state | Context | Tone approach | Example |
|---|---|---|---|
| **Frustrated** | Errors, failures | Empathetic, solution-focused, no blame | "Ingestion failed. Check the extractor logs." |
| **Confused** | First use, complex features | Patient, explanatory | "Connect a data source to bring data into CDF." |
| **Confident** | Routine tasks | Efficient and direct | "Saved." |
| **Cautious** | High-stakes actions | Serious, transparent | "Delete pipeline? All history will be removed." |
| **Successful** | Completions | Positive, proportional | "Pipeline configured. Ingestion starts shortly." |


## How to Write

1. **Identify the audience** — Read `references/cognite-audience.md`. Set reading level and vocabulary based on the persona.
2. **Understand context** — What is the user doing? What emotional state are they in? What are the stakes?
3. **Draft** — Start with what you'd say conversationally. Apply the appropriate pattern. Front-load key information.
4. **Edit — four passes**:
   - **Purposeful**: Does it help the user achieve their goal?
   - **Concise**: Remove every word that doesn't earn its place.
   - **Conversational**: Read aloud. Would you say this?
   - **Clear**: Specific verbs, consistent terminology, unambiguous meaning.
5. **Score (optional)** — Rate each pass 0–10. Use scores to prioritise revisions.
   - 9–10: Excellent · 7–8: Good · 5–6: Needs work · below 5: Rewrite
   - **Example**: "An error occurred while processing your request." → Purposeful: 4/10, Concise: 6/10, Conversational: 5/10, Clear: 5/10 → **Improved**: "We couldn't process your request. Check your connection and try again."


## Accessibility in UX Writing

For translation guidance, high-stress copy, and testing tools, see `references/accessibility-guidelines.md`.

### Screen Reader Optimization
- Label all interactive elements explicitly ("Submit form" not just "Submit")
- Write descriptive link text ("Read pricing details" not "Click here")
- Structure error messages to work with screen readers (error + field label read together)
- Use ARIA labels when visual context isn't available

### Cognitive Accessibility
- Target 8–14 words per sentence (8 words = 100% comprehension, 14 words = 90%)
- Break complex information into scannable chunks
- Use clear headings and logical hierarchy
- Provide consistent, predictable patterns

### Multi-Modal Communication
- Don't rely on color alone to convey meaning
- Pair visual indicators with text ("Error: Field required" with red icon)
- Ensure sufficient color contrast (WCAG AA minimum: 4.5:1)

### Plain Language
- Target 7th–8th grade reading level for general audience
- Define technical terms when first used (unless writing for technical personas)
- Avoid idioms and cultural references


## UX Text Benchmarks

| Element | Target | Maximum |
|---|---|---|
| Buttons / CTAs | 2–4 words | 6 words |
| Titles | 3–6 words, 40 characters | — |
| Tooltips | 10–20 words | 2 sentences |
| Error messages | 12–18 words | — |
| Instructions | 14 words | 20 words |
| Notifications | 10–15 words total | — |
| Line length | 40–60 characters | 70 characters |

**Reading level by persona:**
- Low technical: 7th–8th grade (Flesch-Kincaid)
- Mid technical: 9th–10th grade
- High technical: 10th–11th grade


## Common Mistakes to Avoid

- Generic button labels ("Submit", "OK", "Yes")
- Blaming users in error messages ("invalid input", "illegal character")
- Vague confirmation dialogs ("Are you sure?")
- Dead-end errors with no recovery path ("Something went wrong")
- Passive voice as the default
- Inconsistent terminology for the same concept
- System-oriented language instead of user language
- Relying on color alone for meaning
- Inaccessible link text ("Click here", "Read more")
- Tooltips that just repeat the label


## Quick Reference

- **Sentence case**: "Save your changes" (not "Save Your Changes")
- **Active imperative for buttons**: "Delete pipeline" (not "Pipeline deletion")
- **User-focused**: "See your data faster" (not "We provide fast data access")
- **Specific verbs**: "Delete" (permanent), "Remove" (reversible), "Archive" (hide but keep)
- **Front-loaded**: "Password must be 8 characters" (not "Must be 8 characters for your password")
- **Confirmation**: "Delete pipeline?" (not "Are you sure you want to delete?")
- **Tooltip**: "Asset ID. The unique identifier for this asset." (not "This is the asset ID field")
