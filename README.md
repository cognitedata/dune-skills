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
| **dm-limits-and-best-practices** | CDF Data Modeling API best practices — concurrency, pagination, batching |
| **integrate-file-viewer** | Integrates CogniteFileViewer to preview CDF files (PDFs, images, text) |
| **performance** | Optimizes Dune apps for speed, render counts, and bundle size |
| **security** | Reviews for security issues — credentials, user input, external data |
| **setup-dune-auth** | Migrates React apps to Dune auth or adds DuneAuthProvider |
| **use-aura-design-system** | Implements or migrates to the Aura Design System in Dune/React apps |
| **aura-tokens** | Enforces semantic Aura design tokens via Tailwind |
| **aura-components** | Guides Aura component selection, prevents custom rebuilds |
| **accessibility** | Page-level accessibility — keyboard nav, ARIA, focus management |
| **content-guidelines** | UX writing standards, text patterns, voice and tone |
| **error-validation** | Form validation, loading states, error handling patterns |
| **layout-patterns** | Approved page layouts with responsive specs |
| **design-quality-checklist** | Scores app against 10 Aura design criteria with fix suggestions |

## Contributing

Add a new skill by creating a `skills/<name>/SKILL.md` file with proper frontmatter:

```yaml
---
name: my-skill
description: "When to use this skill..."
allowed-tools: Read, Glob, Grep, Edit, Write
---
```

Or push via the CLI:

```bash
npx @cognite/dune skills push path/to/SKILL.md
```
