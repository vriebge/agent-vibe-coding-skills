# Tech Stack Analysis

## Core Technologies

### Runtime

| Component | Technology | Version | Purpose |
|-----------|------------|---------|---------|
| Server | Bun | 1.0+ | HTTP server, WebSocket, package manager |
| Desktop | Node.js | 18+ | Electron main process |
| Build | Bun | 1.0+ | Fast builds, workspace management |

### Frontend

| Component | Technology | Version | Purpose |
|-----------|------------|---------|---------|
| Framework | React | 18.3 | UI components |
| Build | Vite | 6.2.4 | Fast dev server, HMR |
| Desktop | Electron | 39.2.7 | Cross-platform desktop |
| Packaging | electron-builder | latest | App distribution |
| State | Jotai | 2.x | Atomic state management |
| Styling | Tailwind CSS | 4.1.18 | Utility-first CSS |
| Components | Radix UI | latest | Accessible primitives |
| Rich Text | Tiptap | 3.20 | ProseMirror-based editor |
| Animations | Framer Motion | latest | Declarative animations |
| Icons | Lucide React | latest | Consistent icon set |
| Code Highlighting | Shiki | latest | Syntax highlighting |

### AI/Agent SDKs

| SDK | Purpose |
|-----|---------|
| @anthropic-ai/claude-agent-sdk | Claude AI integration |
| @mariozechner/pi-coding-agent | Pi AI integration |
| @github/copilot-sdk | GitHub Copilot integration |
| @modelcontextprotocol/sdk | MCP standard support |

### Server

| Component | Technology | Purpose |
|-----------|------------|---------|
| HTTP | Bun native | HTTP server |
| WebSocket | Bun native | RPC communication |
| RPC | Custom + MCP | Domain-specific handlers |

### Key Dependencies (Full List)

**AI/Agent:**
- `@anthropic-ai/claude-agent-sdk`
- `@mariozechner/pi-coding-agent`
- `@github/copilot-sdk`
- `@modelcontextprotocol/sdk`

**UI:**
- `react`, `react-dom` (18.3)
- `@radix-ui/react-*` (dialog, dropdown, tabs, etc.)
- `@tiptap/react`, `@tiptap/starter-kit`
- `framer-motion`
- `lucide-react`
- `clsx`, `tailwind-merge` (className utility)
- `class-variance-authority` (component variants)

**Styling:**
- `tailwindcss`, `@tailwindcss/vite`
- `oklch` (color manipulation)

**Markdown:**
- `marked`, `react-markdown`, `remark-gfm`

**Desktop:**
- `electron`, `electron-builder`

**State:**
- `jotai`

**Messaging:**
- `grammy` (Telegram)
- `@whiskeysockets/baileys` (WhatsApp)

**Error Tracking:**
- `@sentry/react`, `@sentry/electron`

**Build:**
- `vite`
- `esbuild` (Electron main/preload)
- `typescript`

**Testing:**
- `bun test`

## Development Tools

| Tool | Purpose |
|------|---------|
| esbuild | Fast JS/TS bundling for Electron main |
| Vite | React app bundling |
| ESLint 9 | Linting |
| Husky 9 | Git hooks |
| TypeScript 5.x | Type safety |

## Tech Stack Evolution

| Phase | Stack | Notes |
|-------|-------|-------|
| Original | React + Webpack | Larger bundles, slower builds |
| Current | React + Vite | Faster builds, better DX |
| Styling | Tailwind v3 → v4 | CSS-first configuration |

## Why These Choices Matter

### Bun Workspaces
- Single package manager for entire monorepo
- Native workspace support (no extra config)
- Fast install and link times

### Vite over Webpack
- Sub-second HMR
- Simpler config
- Native ESM

### Jotai over Redux
- 10x less boilerplate
- Native support for atomFamily (session isolation)
- Built-in TypeScript support

### Tailwind CSS v4
- CSS-first configuration (no tailwind.config.js)
- OKLCH colors for perceptual uniformity
- `@tailwindcss/vite` plugin

### Radix UI Primitives
- Accessible by default
- Unstyled (full control)
- Composeable

### electron-builder
- Multi-platform packaging
- Auto-update support
- Code signing support
