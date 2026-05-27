# Project Structure

## Monorepo Layout

```
craft-agents-oss/
├── apps/                      # Frontend applications
│   ├── electron/             # Desktop Electron app
│   │   ├── src/
│   │   │   ├── main/         # Main process (Node.js)
│   │   │   │   ├── index.ts
│   │   │   │   ├── ipc.ts
│   │   │   │   └── ...
│   │   │   ├── preload/      # Preload scripts
│   │   │   │   └── index.ts
│   │   │   └── renderer/    # React app
│   │   │       ├── main.tsx
│   │   │       ├── App.tsx
│   │   │       ├── atoms/    # Jotai atoms
│   │   │       ├── components/
│   │   │       │   ├── app-shell/
│   │   │       │   ├── ui/
│   │   │       │   ├── browser/
│   │   │       │   ├── settings/
│   │   │       │   └── onboarding/
│   │   │       ├── pages/
│   │   │       ├── hooks/
│   │   │       ├── lib/
│   │   │       └── styles/
│   │   ├── vite.config.ts
│   │   ├── electron-builder.yml
│   │   └── package.json
│   │
│   ├── webui/               # Web UI (served by server)
│   │   ├── src/
│   │   │   └── main.tsx
│   │   ├── vite.config.ts
│   │   └── package.json
│   │
│   ├── viewer/              # Session viewer
│   │   ├── src/
│   │   │   └── main.tsx
│   │   ├── vite.config.ts
│   │   └── package.json
│   │
│   └── cli/                 # CLI tool
│       ├── src/
│       │   └── index.ts
│       └── package.json
│
├── packages/                 # Shared libraries
│   ├── core/                # Type definitions only
│   │   ├── src/
│   │   │   ├── types/
│   │   │   └── utils/
│   │   └── package.json
│   │
│   ├── shared/              # Business logic
│   │   ├── src/
│   │   │   ├── agent/       # AgentBackend factory
│   │   │   ├── auth/        # OAuth flows
│   │   │   ├── config/      # Zod-validated config
│   │   │   ├── credentials/ # Encrypted storage
│   │   │   ├── mcp/        # MCP client pool
│   │   │   ├── protocol/    # RPC channels
│   │   │   ├── sessions/    # Session management
│   │   │   └── workspaces/  # Workspace config
│   │   └── package.json
│   │
│   ├── ui/                  # Shared React components
│   │   ├── src/
│   │   │   ├── components/
│   │   │   │   ├── ui/      # Primitives (Button, Input, etc.)
│   │   │   │   ├── chat/    # Chat components
│   │   │   │   ├── markdown/
│   │   │   │   ├── overlay/
│   │   │   │   ├── code-viewer/
│   │   │   │   ├── terminal/
│   │   │   │   ├── icons/
│   │   │   │   └── annotations/
│   │   │   ├── styles/
│   │   │   └── lib/
│   │   └── package.json
│   │
│   ├── server-core/          # Server infrastructure
│   │   ├── src/
│   │   │   ├── handlers/    # RPC handlers
│   │   │   │   └── rpc/
│   │   │   ├── transport/   # WebSocket transport
│   │   │   ├── sessions/    # Session management
│   │   │   ├── bootstrap/   # Server initialization
│   │   │   └── webui/       # WebUI handler
│   │   └── package.json
│   │
│   ├── server/               # Server entry point
│   │   ├── src/
│   │   │   └── index.ts     # Bun HTTP server
│   │   └── package.json
│   │
│   ├── session-mcp-server/   # MCP server
│   │   ├── src/
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── session-tools-core/   # Tool context
│   │   ├── src/
│   │   │   ├── context.ts
│   │   │   └── handlers/
│   │   └── package.json
│   │
│   ├── pi-agent-server/      # Pi agent subprocess
│   │   ├── src/
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── messaging-gateway/    # Messaging gateway
│   │   ├── src/
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   └── messaging-whatsapp-worker/
│       ├── src/
│       │   └── index.ts
│       └── package.json
│
├── scripts/                  # Build scripts
├── docs/                     # Documentation
├── package.json              # Root - Bun workspace config
├── tsconfig.json            # TypeScript config
├── bunfig.toml             # Bun config
└── Dockerfile.server       # Docker for headless server
```

## Package.json Workspace Config

```json
{
  "workspaces": [
    "apps/*",
    "packages/*"
  ]
}
```

## Key Directories Explained

### apps/electron/src/renderer/

**Purpose:** React application running in Electron renderer process

**Key Files:**
- `main.tsx` - React entry point
- `App.tsx` - Root component
- `atoms/` - Jotai state atoms
- `components/app-shell/` - Main layout (AppShell, TopBar, Sidebar)
- `components/ui/` - Electron-specific UI overrides
- `components/settings/` - Settings panels
- `components/onboarding/` - Onboarding flow
- `pages/` - Route-level components (ChatPage, PreferencesPage)

### packages/shared/src/

**Purpose:** Business logic shared across all apps

**Key Modules:**
- `agent/backend/` - AgentBackend interface + factory
- `auth/` - OAuth 2.0 + PKCE flows
- `config/` - Zod-validated configuration
- `credentials/` - Encrypted API token storage
- `mcp/` - MCP client pool management
- `protocol/` - RPC channel definitions
- `sessions/` - Session CRUD operations
- `workspaces/` - Workspace configuration

### packages/ui/src/components/

**Purpose:** Reusable React components

**Structure:**
- `ui/` - Core primitives (60+ components)
- `chat/` - TurnCard, SessionViewer, etc.
- `markdown/` - Markdown rendering
- `overlay/` - Preview overlays
- `code-viewer/` - Syntax highlighting
- `terminal/` - Terminal output

### packages/server-core/src/

**Purpose:** Reusable server infrastructure

**Structure:**
- `handlers/rpc/` - Domain handlers (auth, sessions, llm-connections, etc.)
- `transport/` - WebSocket transport layer
- `sessions/` - Session management
- `bootstrap/` - Server initialization
- `webui/` - Web UI HTTP handler

## File Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| React Components | PascalCase | `SessionViewer.tsx` |
| Hooks | camelCase with `use` prefix | `useSession.ts` |
| Atoms | camelCase with `Atom` suffix | `sessionAtomFamily.ts` |
| Utilities | camelCase | `formatDate.ts` |
| Config | camelCase | `storage.ts` |
| Types | PascalCase | `SessionMeta` |
| Constants | UPPER_SNAKE_CASE | `MAX_MESSAGE_LENGTH` |
| Directories | kebab-case | `session-tools-core` |

## Import Order

```typescript
// 1. React and hooks
import { useState, useEffect } from 'react';

// 2. External libraries
import { atom } from 'jotai';
import { clsx } from 'clsx';

// 3. Shared packages
import { SessionMeta } from '@craft-agent/core';
import { Button } from '@craft-agent/ui';

// 4. Local components
import { SessionCard } from './components/';
```

## Next Steps

When extracting to template:
1. Identify core patterns per module
2. Create template files with placeholders
3. Document migration steps
4. Provide code examples for each module
