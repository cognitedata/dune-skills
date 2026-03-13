# Using UX Writing Skill with Codex + Figma MCP

Connect this skill to Figma through Codex so you can review and improve UX copy directly from your designs. Perfect for content designers, product designers, and anyone who needs to audit or enhance UX text in Figma mockups.

## What You Can Do

Once connected, you can:
- **Share Figma design links** with Codex and get instant UX writing feedback
- **Audit existing designs** for accessibility, clarity, and tone
- **Review entire flows** for consistency and voice
- **Get specific suggestions** based on the four quality standards (purposeful, concise, conversational, clear)

## Setup: Connect Figma MCP to Codex

### Requirements

- Codex CLI or IDE extension installed
- Figma account
- Internet connection

### Step-by-Step Setup

**Step 1: Configure Codex for MCP**

Open your Codex configuration file at `~/.codex/config.toml` and add these lines:

```toml
[features]
rmcp_client = true

[mcp_servers.figma]
url = "https://mcp.figma.com/mcp"
```

**Step 2: Install Codex CLI**

If you haven't already installed the Codex CLI, install it via npm:

```bash
npm i -g @openai/codex
```

**Step 3: Authenticate with Figma**

Login to Figma via the Codex CLI:

```bash
codex mcp login figma
```

This will open a browser window for authentication. Follow the prompts to allow Codex to access your Figma account.

**Step 4: Restart Your IDE**

If you're using Codex in an IDE (VS Code, etc.), completely restart the IDE to activate the MCP connection.

**Step 5: Verify Connection**

Test the Figma MCP connection:
1. Open a Figma file in your browser
2. Switch to **Dev Mode** (Shift + D)
3. Select any frame or component
4. Copy the section link from Dev Mode
5. Paste the link into Codex in your IDE or CLI

Ask Codex to review the UX copy, and it should be able to access the Figma frame.

---

## How to Use with UX Writing Skill

### Method 1: Review Existing Figma Designs

**Step 1: Get Your Figma Link**

1. Open your design in Figma (web or desktop app)
2. Switch to **Dev Mode** (Shift + D)
3. Select the frame you want to review
4. Copy the section link from Dev Mode

**Step 2: Share with Codex**

In Codex CLI or your IDE, paste the link along with your request:

```
Review the UX copy in this checkout flow:
[Figma section link]

Using the UX Writing Skill, check for:
- Error messages (should be empathetic and actionable)
- Button labels (should be specific verbs)
- Form field labels (should be clear and accessible)
- Overall tone (should be helpful and professional)
```

**Step 3: Get Detailed Feedback**

Codex will:
1. Access the Figma design through MCP
2. Extract all text elements
3. Apply the UX Writing Skill automatically
4. Provide specific, actionable feedback based on the four quality standards

---

## Example Workflows for Content Designers

### 1. Quick UX Audit Before Launch

```
I need to review this feature before launch:
[Figma section link]

Using the UX Writing Skill, audit all copy for:
- Accessibility issues (screen reader compatibility, reading level)
- Sentence length (should be under 20 words)
- Button specificity (no generic "Submit" or "OK" buttons)
- Error message quality (explain problem + solution)
- Voice consistency

Give me a prioritized list of issues to fix.
```

### 2. Voice and Tone Validation

```
Review the tone in these onboarding screens:
[Figma section link]

Our product voice is: helpful, friendly, professional

Check if all the copy matches this voice and suggest improvements
where it doesn't. Use the tone adaptation framework from the UX Writing Skill.
```

### 3. Comprehensive Accessibility Check

```
Audit this form for accessibility:
[Figma section link]

Using accessibility guidelines from the UX Writing Skill, check:
- Screen reader compatibility
- Form labels (visible, not placeholder-only)
- Error messages (descriptive and actionable)
- Plain language (7th-8th grade reading level)
- Link text (descriptive, not "click here")
```

### 4. Before/After Analysis with Scoring

```
Here's my current empty state: [Figma section link]

Using the UX Writing Skill:
1. Score the current copy against the 4 quality standards (purposeful, concise, conversational, clear)
2. Identify specific problems
3. Provide rewritten version with improvements
4. Explain what changed and why
```

### 5. Multi-Platform Consistency Check

```
Compare copy across these platform designs:
- Web: [Figma link 1]
- iOS: [Figma link 2]
- Android: [Figma link 3]

Check for:
- Terminology consistency
- Tone consistency
- Platform-specific conventions (e.g., "tap" vs "click")
- Character count appropriateness for each platform
```

---

## Tips for Best Results

### Be Explicit About Using the UX Writing Skill

For best results, explicitly mention the UX Writing Skill in your prompts:

❌ **Too vague:**
> "Review this design: [link]"

✅ **Better:**
> "Using the UX Writing Skill, review error messages in this form: [link]. Check against the four quality standards."

### Reference Specific Frameworks

The UX Writing Skill includes several frameworks you can call out:

```
Use the tone adaptation framework to suggest appropriate tone for this error state: [link]
```

```
Apply the content usability checklist to score this copy: [link]
```

```
Check this against the accessibility guidelines in the UX Writing Skill
```

### Ask for Specific Patterns

```
Review all buttons in this flow: [link]

Check that they follow the button pattern:
- Active imperative verbs
- [Verb] [object] format
- Specific, not generic
- Under 25 characters
```

### Use Codex's Explicit Skill Invocation

In Codex CLI/IDE, you can explicitly invoke the UX Writing Skill:

```
$ux-writing Review the UX copy in this design: [Figma link]
```

Or use the `/skills` command to select it from the available skills list.

---

## Troubleshooting

### "I don't have access to that Figma file"

**Solutions:**
1. Make sure the Figma file is set to "Anyone with the link can view"
2. Check that you're signed into the same Figma account you authenticated with
3. Try copying the link again from Dev Mode (might have been truncated or expired)
4. Make sure you're using the section link from Dev Mode, not just the file URL

### "MCP connection failed"

**Solutions:**
1. Verify your `~/.codex/config.toml` has the correct configuration
2. Make sure you ran `codex mcp login figma` successfully
3. Check that `rmcp_client = true` is set in the `[features]` section
4. Restart your IDE completely (not just reload window)
5. Try re-authenticating: `codex mcp login figma`

### "Codex doesn't seem to use the UX Writing Skill"

**Solution:**

Be more explicit in your prompt:

```
Using the UX Writing Skill, review this design: [link]

Apply the four quality standards:
1. Purposeful
2. Concise
3. Conversational
4. Clear
```

Or use explicit invocation:

```
$ux-writing analyze the UX copy in this frame: [link]
```

### "The skill isn't installed in Codex"

**Solution:**

Verify the skill is in the correct location:
- **Mac/Linux**: `~/.codex/skills/ux-writing/SKILL.md`
- **Windows**: `%USERPROFILE%\.codex\skills\ux-writing\SKILL.md`

Restart Codex after installation.

Check installed skills using the `/skills` command in Codex.

---

## Example: Complete UX Audit Workflow

Here's a real-world example of conducting a comprehensive UX writing audit:

```
I'm reviewing our checkout flow before launch. Here are the 4 key frames:

1. Cart: [Figma Dev Mode link]
2. Shipping: [Figma Dev Mode link]
3. Payment: [Figma Dev Mode link]
4. Confirmation: [Figma Dev Mode link]

Using the UX Writing Skill, perform a complete audit:

**Check for:**
- All 4 quality standards (purposeful, concise, conversational, clear)
- Accessibility (screen readers, reading level, plain language)
- Error messages (empathetic, actionable, specific)
- Form labels (visible, descriptive, not placeholder-only)
- Button labels (specific verbs, not generic)
- Voice consistency across all screens
- Appropriate tone for context

**Provide:**
1. Overall score (1-10) with explanation
2. Critical issues (must fix before launch)
3. Recommended improvements (nice to have)
4. Rewritten copy for any critical issues
5. Summary of patterns used well

Format as a design review report.
```

---

## Advanced Usage

### Create a Content Pattern Library

```
Review all designs in our product at these key flows: [multiple Figma links]

Extract and document our content patterns:
- Button naming conventions we use
- Error message structure
- Empty state patterns
- Success message patterns
- Voice characteristics (with examples)

Create a pattern library I can share with the team.
```

### Build a Voice Chart from Existing Designs

```
Analyze the copy in these designs: [multiple Figma links]

Using the voice chart template from the UX Writing Skill, create a voice chart showing:
- 3-5 key brand concepts
- Voice characteristics for each
- Do/Don't examples from our actual product
- Tone variations for different contexts
```

### Automated Copy Testing

```
Every week, I'll share new designs with you. For each design:
1. Extract all copy
2. Run it through the content usability checklist
3. Flag anything scoring below 7/10
4. Provide specific fixes
5. Track improvements over time
```

---

## Resources

### Official Documentation
- **Codex Skills**: [developers.openai.com/codex/skills](https://developers.openai.com/codex/skills/)
- **Codex MCP Documentation**: Check Codex docs for MCP server configuration

### UX Writing Skill Documentation
- **Main README**: See the repository README.md for installation and overview
- **SKILL.md**: Core frameworks and patterns
- **Reference Materials**:
  - `references/accessibility-guidelines.md`
  - `references/voice-chart-template.md`
  - `references/content-usability-checklist.md`
  - `references/patterns-detailed.md`

### Community
- Browse the [OpenAI Developer Forum](https://community.openai.com/) for Codex discussions
- Learn about [creating custom skills](https://developers.openai.com/codex/skills/create-skill)

---

## Feedback & Contributions

Have ideas for improving this integration? Found effective workflows? We'd love to hear:
- Real-world examples of UX improvements from Figma reviews via Codex
- Tips for content design teams using Codex + Figma MCP + UX Writing Skill
- Additional prompts or patterns that work well

Open an issue or submit a pull request to share your insights!

---

**Happy designing and writing!** 🎨✍️
