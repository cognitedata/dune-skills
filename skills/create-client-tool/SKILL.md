---
name: create-client-tool
description: "MUST be used whenever creating an AtlasTool (client-side tool) for an Atlas agent. Do NOT manually write AtlasTool definitions or wire them into useAtlasChat — this skill handles the TypeBox schema, execute function, and hook wiring. This includes tools that fetch data, render UI, call APIs, show charts, query local state, or perform any browser-side action. Triggers: AtlasTool, client tool, add tool, create tool, new tool, tool definition, agent tool."
allowed-tools: Read, Glob, Grep, Edit, Write
metadata:
  argument-hint: "[tool-name] [brief description of what it does]"
---

# Create a Client Tool

Scaffold a new `AtlasTool` named **$ARGUMENTS** and wire it into the app.

## Background

Client tools let the Atlas Agent invoke logic that runs in the browser — rendering charts,
querying local state, showing UI panels, triggering navigation, etc. The agent decides when
to call the tool; the app executes it and returns a result.

The flow is:
1. Agent responds with a `clientTool` action
2. The library validates the arguments against the TypeBox schema
3. `execute()` runs in the browser and returns `{ output, details }`
4. `output` (string) is sent back to the agent as the tool result
5. `details` (any shape) is available on `message.toolCalls` for the UI to render

---

## Step 1 — Understand the codebase

Before writing anything, read:

- The file where `useAtlasChat` is called (likely `src/App.tsx`) to find where `tools` is passed
- Any existing tool definitions to match the file/naming conventions

---

## Step 2 — Define the tool

Create the tool as a typed constant. Use `Type` from `@cognite/dune-industrial-components/atlas-agent` to
define the parameters schema — this gives both compile-time types and runtime validation.

```ts
import { Type } from "@cognite/dune-industrial-components/atlas-agent";
import type { AtlasTool } from "@cognite/dune-industrial-components/atlas-agent";

export const myTool: AtlasTool = {
  name: "my_tool",            // snake_case — this is what the agent uses to invoke it
  description:
    "One sentence describing what this tool does and when the agent should call it.",
  parameters: Type.Object({
    exampleParam: Type.String({ description: "What this param is for" }),
    optionalNum: Type.Optional(Type.Number({ description: "..." })),
  }),
  execute: async (args) => {
    // args is fully typed from the schema above
    // Do the work here — call APIs, update state, render UI, etc.
    return {
      output: "Plain text summary sent back to the agent",
      details: {
        // Any structured data you want available in the UI via message.toolCalls
      },
    };
  },
};
```

### TypeBox quick reference

| Schema | Usage |
|---|---|
| `Type.String()` | string |
| `Type.Number()` | number |
| `Type.Boolean()` | boolean |
| `Type.Literal("foo")` | exact value |
| `Type.Union([Type.Literal("a"), Type.Literal("b")])` | enum |
| `Type.Array(Type.String())` | string[] |
| `Type.Object({ ... })` | object |
| `Type.Optional(...)` | mark any field optional |

Always add a `description` to each field — the agent uses these to understand what to pass.

---

## Step 3 — Wire into useAtlasChat

Find the `useAtlasChat` call and add the tool to the `tools` array:

```ts
const { messages, send, ... } = useAtlasChat({
  client: isLoading ? null : sdk,
  agentExternalId: AGENT_EXTERNAL_ID,
  tools: [myTool],   // add here
});
```

---

## Step 4 — Render tool results (if needed)

If the tool returns structured `details`, render them in the message list.
`message.toolCalls` is a `ToolCall[]` — one entry per tool call (client-side and server-side) in call order.

```tsx
{msg.toolCalls?.map((tc, i) => (
  // tc.name    — tool name
  // tc.output  — the string sent back to the agent
  // tc.details — your structured data (cast to your known shape)
  <MyToolOutput key={i} data={tc.details as MyToolDetails} />
))}
```

---

## Done

The agent can now invoke `$ARGUMENTS`. Describe what it does clearly in the `description`
field — the agent relies on that string to decide when and how to call the tool.
