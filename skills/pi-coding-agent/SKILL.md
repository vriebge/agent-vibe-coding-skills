---
name: pi-coding-agent
description: Use when building AI coding agents with pi framework, developing TypeScript extensions, creating skills for on-demand capabilities, or packaging agent tooling for distribution
---

# Pi Coding Agent

## Overview

**Pi** is a minimal terminal coding harness built on TypeScript. It provides a lightweight AI coding agent with extension points for tools, commands, events, and custom UI through a clean plugin architecture.

Core architecture: **CLI → Agent Loop → Provider Abstraction → LLM**

## Core Concepts

### Extension Types

| Type | Purpose | Registration |
|------|---------|--------------|
| **Tools** | LLM-callable functions | `pi.registerTool()` |
| **Commands** | Slash commands like `/cmd` | `pi.registerCommand()` |
| **Events** | Lifecycle hooks | `pi.on("event", handler)` |
| **Providers** | LLM API backends | `pi.registerProvider()` |
| **Skills** | On-demand capability packages | `SKILL.md` in skills/ |
| **Prompts** | Slash command templates | `.md` files in prompts/ |
| **Themes** | TUI styling | JSON in themes/ |

### Session Model

Sessions persist as **JSONL** (JSON Lines) files with tree structure:
- `id`/`parentId` fields enable branching
- Messages flow leaf-to-root for context building
- `SessionManager` handles navigation, branching, compaction

### Event Lifecycle

```
session_start → input → before_agent_start → agent_start
    ↓
turn_loop:
    turn_start → before_provider_request → provider_response
    → tool_execution_start → tool_call → tool_result → tool_execution_end
    ↓
turn_end → agent_end
```

## Extension Development

### Quick Start

```typescript
// extensions/my-extension/index.ts
import type { ExtensionAPI } from "@mariozechner/pi-coding-agent";
import { Type } from "typebox";

export default function (pi: ExtensionAPI) {
  // React to events
  pi.on("session_start", async (_event, ctx) => {
    ctx.ui.notify("Extension loaded!", "info");
  });

  // Register a custom tool
  pi.registerTool({
    name: "greet",
    label: "Greet",
    description: "Greet someone by name",
    parameters: Type.Object({
      name: Type.String({ description: "Name to greet" }),
    }),
    async execute(toolCallId, params, signal, onUpdate, ctx) {
      return { content: [{ type: "text", text: `Hello, ${params.name}!` }], details: {} };
    },
  });

  // Register a command
  pi.registerCommand("hello", {
    description: "Say hello",
    handler: async (args, ctx) => {
      ctx.ui.notify(`Hello ${args || "world"}!`, "info");
    },
  });
}
```

### Extension Locations

- **Global**: `~/.pi/agent/extensions/` or `~/.pi/agent/extensions/*/index.ts`
- **Project**: `.pi/extensions/` or `.pi/extensions/*/index.ts`
- Auto-discovered, hot-reload with `/reload`

### Tool Registration

```typescript
pi.registerTool({
  name: string;
  label: string;
  description: string;
  parameters: TSchema;  // TypeBox schema
  promptSnippet?: string;      // One-line system prompt
  promptGuidelines?: string[]; // When to use this tool
  prepareArguments?: (args) => any;  // Compatibility shim
  async execute(toolCallId, params, signal, onUpdate, ctx) {
    return {
      content: [{ type: "text", text: "..." }],
      details: {},  // Store for branching reconstruction
      terminate?: boolean;  // Skip follow-up LLM call
    };
  },
  renderCall?: (args, theme, context) => TUIComponent;
  renderResult?: (result, options, theme, context) => TUIComponent;
  renderShell?: "self";
});
```

### Event Handling

| Event | Can Block | Purpose |
|-------|-----------|---------|
| `tool_call` | Yes | Block/modify dangerous operations |
| `tool_result` | No | Modify results before display |
| `before_agent_start` | No | Inject messages, modify system prompt |
| `input` | Yes | Transform user input |
| `session_start/shutdown` | No | Lifecycle |

```typescript
// Block dangerous commands
pi.on("tool_call", async (event, ctx) => {
  if (event.toolName === "bash" && event.input.command?.includes("rm -rf")) {
    const ok = await ctx.ui.confirm("Dangerous!", "Allow rm -rf?");
    if (!ok) return { block: true, reason: "Blocked by user" };
  }
});

// Modify results
pi.on("tool_result", async (event, ctx) => {
  // event.toolName, event.result
});
```

### UI Methods (ctx.ui)

| Method | Purpose |
|--------|---------|
| `select(title, options)` | Selection dialog |
| `confirm(title, message)` | Yes/no dialog |
| `input(title, placeholder?)` | Text input |
| `editor(title, prefilled?)` | Multi-line editor |
| `notify(message, type)` | Non-blocking notification |
| `setStatus(key, text)` | Footer status |
| `setWidget(key, content)` | Above/below editor widget |
| `custom(callback)` | Custom TUI component |
| `setEditorComponent(factory)` | Custom editor (vim, emacs) |
| `setTheme(name)` | Switch themes |

### State Management

Store state in tool result `details` for branching support:

```typescript
pi.registerTool({
  name: "my_tool",
  async execute(...) {
    items.push("new item");
    return {
      content: [{ type: "text", text: "Added" }],
      details: { items: [...items] },  // Reconstructed on branch
    };
  },
});
```

## Skill Creation

Skills are self-contained capability packages loaded on-demand via `/skill:name` commands.

### Skill Structure

```
skills/my-skill/
├── SKILL.md              # Required: skill definition
├── scripts/               # Optional: helper scripts
├── references/            # Optional: documentation
└── assets/                # Optional: images, etc.
```

### SKILL.md Format

```markdown
---
name: my-skill-with-hyphens
description: Use when [specific triggering conditions and symptoms]
---

# My Skill

## Overview
One sentence on core principle.

## When to Use
Bullet list with symptoms and use cases.

## Core Pattern
Before/after code comparison.

## Quick Reference
Table or bullets for scanning.
```

### Frontmatter Requirements

| Field | Required | Constraints |
|-------|----------|-------------|
| `name` | Yes | Max 64 chars, lowercase letters, numbers, hyphens only |
| `description` | Yes | Max 1024 chars, third person, start with "Use when..." |

### Skill Discovery

Pi scans these locations:
- Global: `~/.pi/agent/skills/`, `~/.agents/skills/`
- Project: `.pi/skills/`, `.agents/skills/`
- Packages: `skills/` directories or `pi.skills` in package.json
- CLI: `--skill <path>`

**Progressive disclosure**: Descriptions always in context, full instructions load on-demand.

## Package Creation

Packages bundle extensions, skills, prompts, and themes for npm distribution.

### Package Structure

**Option A: pi manifest in package.json**
```json
{
  "name": "@my/pi-package",
  "pi": {
    "extensions": ["dist/"],
    "skills": ["skills/"],
    "prompts": ["prompts/"],
    "themes": ["themes/"]
  }
}
```

**Option B: Conventional directories**
```
my-pi-package/
├── extensions/           # .ts files
├── skills/                # SKILL.md files
├── prompts/               # .md templates
└── themes/                # JSON theme files
```

### Package Installation

```bash
pi install npm:@my/package@1.0.0
pi install git:github.com/user/repo@v1
pi install /local/path
```

### Scope

| Scope | Location | Priority |
|-------|----------|----------|
| Global | `~/.pi/agent/settings.json` | Lower |
| Project | `.pi/settings.json` | Higher |

## Provider Registration

### Override Built-in Provider

```json
{
  "providers": {
    "anthropic": {
      "baseUrl": "https://my-proxy.com/v1",
      "headers": { "X-Custom": "value" }
    }
  }
}
```

### New Provider

```typescript
pi.registerProvider("my-provider", {
  name: "My Provider",
  baseUrl: "https://api.example.com",
  apiKey: "key-or-env-var-name",
  api: "openai-completions",  // API type
  models: [
    {
      id: "my-model",
      name: "My Model",
      reasoning: false,
      input: ["text"],
      contextWindow: 128000,
      maxTokens: 16384,
    },
  ],
  oauth?: {  // Optional OAuth flow
    login: async () => {},
    refreshToken: async () => {},
    getApiKey: async () => {},
  },
});
```

### API Types

| API Type | Use Case |
|----------|----------|
| `anthropic-messages` | Anthropic Claude API |
| `openai-completions` | OpenAI Chat Completions |
| `openai-responses` | OpenAI Responses API |
| `google-generative-ai` | Google Generative AI |
| `google-vertex` | Google Vertex AI |
| `bedrock-converse-stream` | AWS Bedrock |
| `mistral-conversations` | Mistral SDK |

## RPC Mode

Headless operation via JSONL stdin/stdout.

### Start RPC Mode

```bash
pi --mode rpc [options]
```

### Core Commands (stdin → stdout)

```typescript
// Prompt
{ "type": "prompt", "content": "Hello", "id": "req-1" }

// Abort
{ "type": "abort", "id": "req-1" }

// Bash
{ "type": "bash", "command": "ls -la" }

// Get state
{ "type": "get_state" }

// Set model
{ "type": "set_model", "model": "claude-sonnet-4-20250514" }
```

### Event Types (stdout)

```
agent_start → turn_start → message_start → [streaming] → message_end → turn_end → agent_end
tool_execution_start → tool_execution_update → tool_execution_end
queue_update, compaction_start/end, auto_retry_start/end
extension_ui_request (blocks for extension_ui_response)
```

## Prompt Templates

Expand into full prompts via `/name` slash commands.

### Location

- Global: `~/.pi/agent/prompts/*.md`
- Project: `.pi/prompts/*.md`
- Packages: `prompts/` or `pi.prompts` in package.json

### Format

```markdown
---
description: Review staged git changes
---
Review the staged changes (`git diff --cached`). Focus on:
- Bugs and logic errors
- Security issues
```

Filename becomes command (`review.md` → `/review`).

### Arguments

| Syntax | Expansion |
|--------|------------|
| `$1`, `$2` | Positional args |
| `$@`, `$ARGUMENTS` | All args |
| `${@:N}` | Args from position N |
| `${@:N:L}` | L args starting at N |

## Quick Reference

### Extension Locations
```
~/.pi/agent/extensions/     # Global
~/.pi/agent/skills/          # Global skills
~/.pi/agent/prompts/         # Global templates
.pi/extensions/             # Project-local
.pi/skills/                 # Project skills
```

### Key Registration Methods
```typescript
pi.registerTool({})      // LLM-callable function
pi.registerCommand({})   // Slash command
pi.registerProvider({})  // LLM backend
pi.on("event", fn)       // Lifecycle hook
```

### Session Management
```typescript
ctx.sessionManager.getEntries()     // All entries
ctx.sessionManager.getBranch()      // Current branch
ctx.sessionManager.branch()          // Create branch
ctx.sessionManager.buildSessionContext()  // Leaf-to-root messages
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| No truncation in tool output | Use `truncateHead()` from pi-coding-agent |
| State not branching | Store in `details` object, not closure |
| Blocking non-blocking events | Return `{ block: true }` only for blockable events |
| Hardcoded paths | Use `src/config.ts` for package assets |
| No error signaling | Throw `Error` from `execute()` for failures |

## Resources

- [pi-mono repo](https://github.com/badlogic/pi-mono)
- [Agent Skills spec](https://agentskills.io/specification)
- [Anthropic Skills](https://github.com/anthropics/skills)