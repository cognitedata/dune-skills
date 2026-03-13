# /design-review

Run a full design quality review of this project against Aura
design standards. Score 10 criteria, generate a report with
actionable fix suggestions, and track scores over time.

## Instructions

1. Read the design review skill:
   `.cursor/skills/design-review/SKILL.md`

2. Read the workflow: `.cursor/skills/design-review/workflow.md`

3. Check for prior reviews in `.design-reviews/history.json`

4. Scan the codebase following the workflow:
   - Use grep/search for CODE and GREP verifiable criteria
   - For each finding, note the file, line, and specific issue
   - Classify findings as CODE, GREP, VISUAL, or MANUAL

4b. Visual verification (tool-dependent): If visual verification tools
    are available in your environment, perform visual checks for criteria
    5, 7, 9, 10. Otherwise, continue with code-only analysis and mark
    those criteria as "(code-level only)". 
    
    Available visual verification approaches:
    - Screenshot comparison tools
    - Browser automation capabilities  
    - Manual spot-checking if development server is running
    
    Focus on responsive breakpoints, hover states, keyboard navigation,
    and accessibility features that are difficult to verify through
    code analysis alone.

5. Score each of the 10 criteria (1-5) using the rubrics.
   Before listing findings, apply pattern grouping per the
   `<pattern-grouping>` section in the design review skill.
   Aggregate findings that share the same root cause into
   a single [PATTERN] entry with instance count and
   consolidated fix prompt.

6. Auto-fix pass: Apply all auto-fixable findings per
   `<auto-fix-rules>` in the design review skill. Group edits
   by file. Re-verify affected criteria and re-score.

7. Generate the report in the format specified by the design review
   skill, including:
   - Overall score and level
   - Pattern-grouped findings with fix prompts
   - Pre-fix vs post-fix score comparison
   - Score comparison to previous review (if exists)
   - Applied fixes list
   - Priority fixes
   - Items needing human review

8. Save results:
   - Create `.design-reviews/` directory if needed
   - Write full report to `.design-reviews/YYYY-MM-DD_review.md`
   - Update `.design-reviews/history.json` with post-fix scores

9. Present the overall score, top 3 priority fixes, and
   score trends to the builder

## Skills referenced

This command cross-references these skills during evaluation:
- `.cursor/skills/aura-tokens/SKILL.md`
- `.cursor/skills/aura-components/SKILL.md`
- `.cursor/skills/content-guidelines/SKILL.md`
- `.cursor/skills/layout-patterns/SKILL.md`
- `.cursor/skills/accessibility/SKILL.md`
- `.cursor/skills/error-validation/SKILL.md`
