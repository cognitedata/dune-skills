# Accessibility Guidelines for UX Writing

Supplement to the core accessibility principles in `SKILL.md`. This file covers three areas not handled there: writing for translation, high-stress context copy, and testing tools.

> For code-level accessibility (keyboard navigation, ARIA attributes, focus management),
> see the **design-accessibility** skill.

---

## Writing for Translation

### Keep It Simple
- Short sentences translate more accurately
- Simple grammar reduces translation errors
- Common words have clearer equivalents

### Avoid Culturally-Specific References
- ❌ "Home run" (baseball reference)
- ❌ "The ball is in your court" (idiom)
- ❌ "During the holidays" (varies by culture)
- ✅ Use universal concepts

### Plan for Text Expansion
Text expands in translation — design UI containers and buttons to accommodate:

| Language | Expansion |
|---|---|
| German | +30–40% |
| French / Spanish | +15–20% |
| Italian / Portuguese | +20–25% |

**Design implications:**
- Buttons: allow for 150–200% text expansion
- Titles: plan for 130–150% expansion
- Character limits: test with the longest likely translation

### Gender-Neutral Language
- Use "they/them" for unknown subjects
- Avoid gendered job titles
  - ❌ "Policeman" → ✅ "Police officer"
  - ❌ "Stewardess" → ✅ "Flight attendant"
- Structure sentences to avoid gender assumptions

---

## High-Stress Context Accessibility

Users experiencing stress, frustration, or urgency have reduced cognitive capacity. Adjust writing accordingly.

### Error Messages
- **Be immediately clear** — State the problem in the first 5 words
- **Provide quick recovery** — One-step solution when possible
- **Avoid blame** — Never use judgmental language
- **Stay calm** — Reassuring without being condescending

### Time-Sensitive Actions
- **Specific deadlines** — "5 minutes remaining", not "soon"
- **Visible countdown** — State the time constraint explicitly
- **Obvious CTA** — Single, prominent action

### High-Stakes Decisions
- **Transparent consequences** — "You'll lose all pipeline history"
- **Reversibility** — State clearly if action can be undone
- **Easy exit** — Clear "Cancel" always present

---

## Testing for Accessibility

### Automated Tools
- **WAVE**: [wave.webaim.org](https://wave.webaim.org/) — web accessibility evaluation
- **axe DevTools**: browser extension for accessibility testing
- **Lighthouse**: built into Chrome DevTools

### Manual Tests

**Screen reader test**
- Enable VoiceOver (Mac) or NVDA (Windows)
- Navigate using keyboard only
- Verify all content is announced meaningfully
- Check that error messages are clear when announced alongside field labels

**Readability test**
- [Hemingway Editor](http://hemingwayapp.com/) — highlights complex sentences
- [Readable.com](https://readable.com/) — multiple readability scores
- Microsoft Word — built-in Flesch-Kincaid scoring

**Color contrast test**
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- Body text: 4.5:1 minimum (WCAG AA)
- Large text and UI elements: 3:1 minimum

**Keyboard navigation test**
- Unplug your mouse; complete all tasks using only keyboard
- Verify all interactive elements are reachable
- Check that focus order is logical

---

## Quick Checklist — Before Publishing Any UX Text

- [ ] All interactive elements have clear, descriptive labels
- [ ] Links describe destination ("View pricing" not "Click here")
- [ ] Error messages are specific and actionable
- [ ] Color is not the only indicator of meaning
- [ ] Text has sufficient contrast (4.5:1 minimum)
- [ ] Sentences average 15–20 words or fewer
- [ ] Reading level matches the target persona
- [ ] No idioms, metaphors, or cultural references
- [ ] Required fields are marked with more than just color
- [ ] Form instructions appear before input fields
- [ ] Success and error states include text, not just icons
