# Dune Skills

Agent-agnostic AI skills for [Dune](https://github.com/cognitedata/dune) apps. Works with Claude Code, Cursor, Copilot, and any agent that supports the [skills format](https://github.com/anthropics/skill-spec).

## Install

Pull all skills into your Dune app:

```bash
npx @cognite/dune skills pull
```

## Available Skills

| Skill | Description |
|-------|-------------|
| **create-client-tool** | Scaffolds an `AtlasTool` and wires it into `useAtlasChat` |
| **integrate-atlas-chat** | Adds streaming Atlas Agent chat UI to a Dune app |
| **setup-python-tools** | Adds Pyodide-based Python tool execution |

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
