---
name: context-sharing
description: >-
  Builds a commit-time context window file that captures what changed, why it changed, decisions
  made with rationale, open questions, and reproducibility notes. Use when preparing a commit,
  writing a commit message, summarizing work for handoff, or documenting decisions in any
  Git-based project (including Dune apps). Triggers: commit message, handoff summary, work
  summary, context window, reproducibility notes, before commit.
allowed-tools: Read, Glob, Grep, Shell, Write
metadata:
  argument-hint: ""
---

# Context sharing (commit context window)

Create a reproducible context window file before each meaningful commit so collaborators can understand intent, decisions, and verification steps without replaying the full chat.

## When to run

Run this skill when:

- The user is about to commit
- The user asks for a commit message
- The user asks for a handoff summary
- The work includes architectural or tradeoff decisions

If no code changed, still create a short context file explaining why no commit is needed.

## Output contract

Write one file per commit preparation:

- **Path:** default `docs/context-windows/` (use another documented folder if the repo standardizes handoffs elsewhere, e.g. `docs/handoff/`).
- **Filename:** `YYYY-MM-DD_HHMM_<branch>_<slug>.md`
- **Time:** local 24-hour clock
- **Slug:** short kebab-case topic (for example: `seed-upsert-retries`)

Use the exact section order below:

1. Objective
2. Scope
3. Changes Made
4. Decisions and Rationale
5. Risks and Mitigations
6. Validation Performed
7. Reproduction Steps
8. Open Questions
9. Next Suggested Actions

## Required inputs

Gather evidence from repository state (do not rely on memory only):

1. `git status --short`
2. `git diff --stat` and relevant `git diff`
3. `git log --oneline -10` for recent context
4. Any user constraints stated in this session

If evidence conflicts with earlier assumptions, trust repository state and note the correction.

## Workflow (deterministic checklist)

Copy and complete this checklist internally before finalizing:

- [ ] Capture current branch and timestamp
- [ ] Capture changed files and classify by type (feature, fix, refactor, docs, test, chore)
- [ ] Summarize user intent in one sentence
- [ ] Record decisions with explicit alternatives considered
- [ ] Record at least one risk (or state "No material risks identified")
- [ ] Record what was validated and what was not validated
- [ ] Add exact reproduction commands
- [ ] Add open questions or explicitly write "None"
- [ ] Save file under the chosen context-windows (or team) directory

## Content rules

- Be concrete and verifiable; reference real files, commands, and outcomes.
- Prefer short bullets over long prose.
- Do not include secrets, tokens, or credentials.
- Separate facts from assumptions.
- If tests were not run, say so explicitly.
- If lints were not run, say so explicitly.
- If there are unrelated workspace changes, call them out under Scope.

## Section guidance

### 1) Objective

One sentence: what problem this work solves now.

### 2) Scope

- In-scope files/directories touched
- Explicit out-of-scope items
- Any known unrelated dirty files

### 3) Changes Made

For each grouped change:

- What changed
- Why it changed
- Primary files involved

### 4) Decisions and Rationale

For each key decision:

- Decision
- Alternatives considered
- Why chosen now
- Follow-up trigger (what would make us revisit)

### 5) Risks and Mitigations

List likely failure modes and how they were reduced.

### 6) Validation Performed

- Commands run and outcomes
- Manual checks
- Explicit gaps (not tested)

### 7) Reproduction Steps

Minimal ordered commands another collaborator can run to reproduce the result.

### 8) Open Questions

Unknowns that block confidence or future work.

### 9) Next Suggested Actions

Up to 5 concrete next steps.

## Template

Use this exact scaffold:

```markdown
# Context Window: <short title>

## Objective
<one sentence>

## Scope
- In scope:
- Out of scope:
- Unrelated workspace changes:

## Changes Made
- <change group>: <what + why> (`path/a`, `path/b`)

## Decisions and Rationale
- Decision: <decision>
  - Alternatives: <alt 1>, <alt 2>
  - Rationale: <why>
  - Revisit if: <condition>

## Risks and Mitigations
- Risk: <risk>
  - Mitigation: <mitigation>

## Validation Performed
- `command`: <result>
- Manual: <result>
- Gaps: <what was not validated>

## Reproduction Steps
1. `<command>`
2. `<command>`
3. `<command>`

## Open Questions
- <question or "None">

## Next Suggested Actions
1. <action>
2. <action>
```

## Quality bar

The file is complete only if a collaborator can answer:

- What changed?
- Why this approach?
- How to verify it?
- What is still uncertain?

If any answer is missing, revise before finishing.

## Additional resource

For reproducibility and handoff standards, read [reference.md](reference.md).
