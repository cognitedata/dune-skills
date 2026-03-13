# Tool Configuration

This folder contains per-tool adapter configs for consuming the
design skills. Each tool has its own subfolder.

## Cursor

**Option A — Pull skills via CLI:**

```bash
npx @cognite/dune skills pull
```

Skills are installed to `.cursor/skills/` in the target project and auto-discovered by Cursor.

**Option B — Symlink skills globally:**

```bash
# Run from dune-skills/
for skill in skills/design/*/; do
  name=$(basename "$skill")
  [ "$name" = "shared" ] && continue
  [ "$name" = "tool-configs" ] && continue
  ln -sf "$(cd "$skill" && pwd)" "$HOME/.cursor/skills/$name"
done
```

**Rules**: Copy or symlink the `.mdc` rule files into
`<project>/.cursor/rules/` for the project you want to review.

```bash
# Run from the target project root
mkdir -p .cursor/rules
ln -sf /path/to/dune-skills/skills/design/tool-configs/cursor/design-review.mdc .cursor/rules/design-review.mdc
```

## Claude Code

**Commands**: Symlink the command files into
`<project>/.claude/commands/`.

```bash
# Run from the target project root
mkdir -p .claude/commands
ln -sf /path/to/dune-skills/skills/design/tool-configs/claude-code/design-review.md .claude/commands/design-review.md
```

Then use `/design-review` in Claude Code to trigger the review.

## Adding a New Tool

1. Create a new subfolder under `tool-configs/` (e.g., `codex/`)
2. Write the adapter that references `design-review/workflow.md`
   for the shared workflow
3. Document the install steps in this README
