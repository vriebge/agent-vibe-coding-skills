# Architecture Philosophy

## Core Design Principles

### 1. Separation of Concerns

Craft Agents maintains strict separation between:
- **Apps layer**: Applications (electron, webui, viewer, cli)
- **Packages layer**: Shared libraries (core, shared, ui, server-core, server)
- **Core** is dependency-light; **Shared** contains business logic

**Rationale:** Allows multiple frontends (Electron, Web) to share business logic without circular dependencies.

### 2. IPC-Based Communication

Electron uses structured IPC channels for main/renderer communication:
- Type-safe message passing
- Async request/response pattern
- Event-based updates for streaming

**Key Pattern:**
```typescript
// In preload: expose IPC via contextBridge
contextBridge.exposeInMainWorld('electronAPI', {
  send: (channel, data) => ipcRenderer.send(channel, data),
  invoke: (channel, data) => ipcRenderer.invoke(channel, data),
  on: (channel, callback) => ipcRenderer.on(channel, callback),
});
```

### 3. Session Isolation

Sessions are first-class citizens with isolated state:
- Each session has its own atom (atomFamily)
- Streaming in Session A does not trigger re-renders in Session B
- Lazy loading: metadata first, messages on-demand

**Memory Impact:** ~50MB vs ~500MB for 300+ sessions.

### 4. Multi-Provider Abstraction

Agent backends are abstracted via `AgentBackend` interface:
```typescript
interface AgentBackend {
  createSession(): Session;
  sendMessage(sessionId: string, message: Message): Promise<Response>;
  streamMessage(sessionId: string, message: Message): AsyncIterable<Response>;
}
```

**Factory Pattern:** `createAgent(provider)` returns appropriate backend.

### 5. MCP Standard Integration

Tools are exposed via Model Context Protocol:
- Session-scoped MCP servers
- HTTP and stdio transport support
- Tool context abstraction for multi-environment execution

### 6. Configuration-Driven Design

All hyperparameters via config objects (not hardcoded):
```typescript
@register_model('MyModel')
class MyModel(nn.Module):
  def __init__(self, cfg: Config):
    super().__init__()
    self.hidden_dim = cfg.model.hidden_dim
```

## Module Relationships

```
┌─────────────────────────────────────────────────────────────┐
│                        APPS                                 │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐       │
│  │electron │  │  webui  │  │ viewer  │  │   cli   │       │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘       │
└───────┼───────────┼───────────┼───────────┼──────────────┘
        │           │           │           │
        ▼           ▼           ▼           ▼
┌─────────────────────────────────────────────────────────────┐
│                      PACKAGES                                │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │  core   │  │ shared  │  │   ui    │  │server-  │        │
│  │ (types) │  │ (logic) │  │ (React) │  │  core   │        │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘        │
└─────────────────────────────────────────────────────────────┘
```

## Key Architectural Decisions

| Decision | Alternative | Why Chosen |
|----------|-------------|------------|
| Bun workspaces | Lerna/Yarn workspaces | Native Bun support, fast installs |
| Jotai atomFamily | Redux/Zustand | Session isolation without boilerplate |
| Tailwind CSS v4 | CSS Modules | CSS-first config, OKLCH colors |
| WebSocket RPC | REST API | Bidirectional streaming, real-time updates |
| MCP SDK | Custom protocol | Standard, multi-provider support |
| electron-builder | electron-forge | Simpler config, better packaging |

## Best Practices

1. **Keep core lightweight** - Only types and utilities
2. **Session isolation first** - Prevent cross-session state pollution
3. **Config-driven models** - All hyperparameters from config objects
4. **Factory pattern for agents** - Easy provider switching
5. **Type-safe IPC** - Use TypeScript interfaces for IPC channels
