---
name: craft-agent-template
description: Use when extracting architecture patterns from Craft Agents OSS, building Electron-based AI agent applications, or migrating a monorepo with Bun workspaces, React + Vite frontend, and Bun HTTP server to a fullstack template.
version: 0.2.0
---

# Craft Agents Template Skill

Extract architecture patterns from Craft Agents OSS to build Electron-based AI agent applications.

## Purpose

This skill provides guidance for extracting Craft Agents' architecture into a reusable fullstack development template. It covers project structure, tech stack decisions, and key patterns for building Electron + React + Bun Server applications with AI agent capabilities.

## When to Use

Use this skill when:
- Extracting architecture patterns from Craft Agents OSS for a new project
- Building an Electron-based AI assistant application
- Migrating a monorepo with Bun workspaces to a fullstack template
- Understanding the AppShell layout, Jotai state management, or MCP integration
- Setting up Tailwind CSS v4 with OKLCH color system
- Configuring WebSocket RPC communication between frontend and backend

## Core Concepts

### 1. Workspace Monorepo Structure

Craft Agents uses Bun workspaces to manage multiple apps and packages:

```
/
├── apps/                    # Frontend applications
│   ├── electron/           # Desktop Electron app
│   ├── webui/             # Web UI served by server
│   ├── viewer/             # Session viewer
│   └── cli/                # CLI tool
├── packages/               # Shared libraries
│   ├── core/              # Type definitions, lightweight
│   ├── shared/            # Business logic (auth, config, credentials)
│   ├── ui/                # Shared React components
│   ├── server-core/       # Server infrastructure
│   └── server/            # Bun HTTP server entry
```

**Principle:** Keep `core` dependency-light; put business logic in `shared`.

### 2. Electron Architecture

Separate main process (Node.js) from renderer process (React + Vite):

- **Main Process:** electron/main/index.ts - Window management, IPC handlers, native APIs
- **Preload:** electron/preload/ - Secure bridge between main and renderer
- **Renderer:** electron/renderer/ - React app with AppShell layout

**IPC Pattern:** Use structured IPC channels for type-safe communication.

### 3. State Management with Jotai

Session isolation via atomFamily:

```typescript
// sessions.ts
const sessionAtomFamily = atomFamily((sessionId: string) => atom(initialState));
const activeSessionIdAtom = atom<string | null>(null);
```

**Principle:** Each session has isolated state to prevent streaming updates in one session from affecting others.

### 4. UI Component Architecture

Shared UI components in `packages/ui/`:
- Core primitives (Button, Input, Dialog)
- Chat components (TurnCard, SessionViewer)
- Markdown rendering
- Tiptap rich text editor integration

**Styling:** Tailwind CSS v4 with CSS-first configuration, OKLCH colors, dark mode via `.dark` class.

### 5. Bun HTTP Server + WebSocket RPC

Server architecture:
- Bun HTTP server with WebSocket upgrade for RPC
- RPC handlers organized by domain (sessions, auth, llm-connections)
- Optional WebUI serving on same port

### 6. MCP Integration

Model Context Protocol support via `@modelcontextprotocol/sdk`:
- Session-scoped MCP servers
- HTTP and stdio transport support
- Tool definitions and handlers

## Progressive Loading Structure

This skill supports **progressive loading** — reference only what you need for your task:

### Feature: New Project Setup
Start here for new projects:
- `references/architecture-philosophy.md` - Core principles
- `references/project-structure.md` - Package organization
- `references/migration-guide.md` - Step-by-step setup

### Feature: UI/Theme Development
Focus on visual design and components:
- `references/ui-design.md` - **Primary reference** for styling
- `references/ui-design-extended.md` - Color system & tokens
- `references/ui-components.md` - Component patterns

### Feature: Electron Development
Desktop-specific development:
- `references/electron-architecture.md` - Main/renderer separation
- `references/ipc-patterns.md` - Secure IPC communication

### Feature: Backend Development
Server and API development:
- `references/bun-server.md` - HTTP server & WebSocket RPC
- `references/mcp-integration.md` - Tool extension via MCP

### Feature: State Management
Jotai atomic state patterns:
- `references/react-patterns.md` - AppShell layout & Jotai

## Reference Files

### Architecture & Structure
- **`references/architecture-philosophy.md`** - Core design principles
- **`references/tech-stack.md`** - Complete tech stack analysis
- **`references/project-structure.md`** - Monorepo structure

### UI/Theme (Primary: ui-design.md)
- **`references/ui-design.md`** - Tailwind v4, OKLCH colors, dark mode
- **`references/ui-design-extended.md`** - Color tokens, theme system
- **`references/ui-components.md`** - Shared component library

### Frontend
- **`references/electron-architecture.md`** - Electron patterns
- **`references/react-patterns.md`** - AppShell, Jotai state

### Backend
- **`references/bun-server.md`** - Bun HTTP server
- **`references/mcp-integration.md`** - MCP SDK integration

### Migration
- **`references/migration-guide.md`** - Migration best practices

## Example Usage

### Building a New Electron App

1. Read architecture philosophy to understand core patterns
2. Study electron-architecture.md for main/renderer separation
3. Reference ui-design.md for styling setup
4. Follow migration-guide.md for project setup

### Adding a New Shared Package

1. Read project-structure.md for package organization
2. Follow existing patterns in `packages/shared/`
3. Use Zod schemas for config validation
4. Register in workspace package.json

### Understanding MCP Integration

1. Read mcp-integration.md for SDK usage
2. Study session-tools-core for tool context pattern
3. Reference session-mcp-server for implementation

## Key Patterns

| Pattern | Purpose | Location |
|---------|---------|----------|
| Workspace Monorepo | Manage multi-package dependencies | Root package.json |
| Session Isolation | Prevent cross-session re-renders | apps/electron/renderer/atoms/ |
| Agent Backend Factory | Multi-provider AI abstraction | packages/shared/src/agent/backend/ |
| CSS OKLCH Colors | Perceptually uniform theming | packages/ui/src/styles/ |
| RPC Handler Organization | Domain-driven server handlers | packages/server-core/src/handlers/rpc/ |

## Quick Reference

**Tech Stack:**
- Runtime: Bun 1.0+ (server), Node.js 18+ (Electron)
- Frontend: React 18, Vite 6, Tailwind CSS v4
- State: Jotai 2.x with atomFamily
- Desktop: Electron 39 + electron-builder
- AI: Anthropic SDK, GitHub Copilot SDK, Pi SDK
- Protocol: MCP SDK for tool extension

**Build Commands:**
```bash
bun run electron:dev      # Start Electron in dev mode
bun run server:dev        # Start server in dev mode
bun run webui:dev         # Start web UI dev server
bun run electron:dist:mac # Build macOS app
```

**Key Files:**
- Electron main: `apps/electron/src/main/index.ts`
- Electron renderer entry: `apps/electron/src/renderer/main.tsx`
- Electron renderer styles: `apps/electron/src/renderer/index.css`
- Server entry: `packages/server/src/index.ts`
- Shared UI components: `packages/ui/src/components/`
- UI styles (Tailwind v4): `apps/electron/src/renderer/index.css`
- Theme tokens: `apps/electron/resources/themes/*.json`
