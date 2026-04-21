# Dune Skills

Agent-agnostic AI skills for [Dune](https://github.com/cognitedata/dune) apps. Works with Claude Code, Cursor, Copilot, and any agent that supports the [skills format](https://github.com/anthropics/skill-spec).

## Install

Pull all skills into your Dune app:

```bash
npx @cognite/dune skills pull
```

Pull a specific skill:

```bash
npx @cognite/dune skills pull --skill create-client-tool
```

## Available Skills

| Skill | Description |
|-------|-------------|
| **create-client-tool** | Scaffolds an `AtlasTool` and wires it into `useAtlasChat` |
| **integrate-atlas-chat** | Adds streaming Atlas Agent chat UI to a Dune app |
| **setup-python-tools** | Adds Pyodide-based Python tool execution |
| **code-quality** | Reviews Dune apps for code quality, maintainability, and clean code issues |
| **correctness-and-error-handling** | Reviews for bugs, missing error states, unhandled rejections, and edge cases |
| **context-sharing** | Commit-time context window / handoff doc — what changed, why, decisions, validation, reproduction |
| **dm-limits-and-best-practices** | CDF Data Modeling API best practices — concurrency, pagination, batching |
| **dune-app-review** | Runs the official Dune app platform review flow against a local app workspace and writes artifacts under `reviews/dune-app-review/feedback-round-N/` |
| **integrate-file-viewer** | Integrates CogniteFileViewer to preview CDF files (PDFs, images, text) |
| **performance** | Optimizes Dune apps for speed, render counts, and bundle size |
| **security** | Reviews for security issues — credentials, user input, external data |
| **setup-dune-auth** | Migrates React apps to Dune auth or adds DuneAuthProvider |
| **design** | Aura UI — components and tokens, layouts, UX copy, forms/async feedback, accessibility (`skills/design/`) |

## Contributing

Add a new skill by creating a `skills/<name>/SKILL.md` file with proper frontmatter:

```yaml
---
name: my-skill
description: "When to use this skill..."
allowed-tools: Read, Glob, Grep, Edit, Write
---
```

Consolidated Aura guidance uses the **`design`** skill (`skills/design/SKILL.md`). Older per-topic `design-*` skills were merged into that folder.

**Authoring style**: Use `## Section` markdown headings for structural sections like Role, When to use, and Setup. XML tags (e.g., `<rubric>`, `<example>`) are acceptable where they serve a machine-parsing purpose.

Or push via the CLI:

```bash
npx @cognite/dune skills push path/to/SKILL.md
```
