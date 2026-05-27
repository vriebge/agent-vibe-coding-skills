# Bun Server Architecture

## Overview

Craft Agents uses Bun as the HTTP server runtime with WebSocket support for RPC communication.

## Server Entry Point

**Location:** `packages/server/src/index.ts`

```typescript
import { BunServer } from './server';

const server = new BunServer({
  port: parseInt(process.env.CRAFT_RPC_PORT || '8080'),
  token: process.env.CRAFT_SERVER_TOKEN,
});

server.start();
```

## HTTP Server Setup

### Basic HTTP Server

```typescript
import type { Server as HTTPServer } from 'bun';
import { randomUUID } from 'crypto';

export class BunServer {
  private server: HTTPServer;
  private wss: Map<string, WebSocket> = new Map();

  constructor(private config: ServerConfig) {
    this.server = Bun.serve({
      port: this.config.port,
      fetch: (req, server) => this.handleRequest(req, server),
      websocket: {
        open: (ws) => this.handleOpen(ws),
        message: (ws, msg) => this.handleMessage(ws, msg),
        close: (ws, code, reason) => this.handleClose(ws, code, reason),
      },
    });
  }

  private handleRequest(req: Request, server: HTTPServer): Response {
    const url = new URL(req.url);

    // Health check
    if (url.pathname === '/health') {
      return Response.json({ status: 'ok' });
    }

    // WebSocket upgrade
    if (server.upgrade(req)) {
      return;
    }

    return Response.json({ error: 'Not found' }, { status: 404 });
  }
}
```

### WebSocket RPC Handler

```typescript
interface RPCMessage {
  id: string;
  method: string;
  params?: any;
}

interface RPCResponse {
  id: string;
  result?: any;
  error?: { code: number; message: string };
}

export class BunServer {
  private handleMessage(ws: WebSocket, msg: string | Buffer) {
    const message: RPCMessage = JSON.parse(msg.toString());

    switch (message.method) {
      case 'sessions:list':
        this.handleListSessions(ws, message);
        break;
      case 'sessions:create':
        this.handleCreateSession(ws, message);
        break;
      // ...
    }
  }

  private sendResponse(ws: WebSocket, response: RPCResponse) {
    ws.send(JSON.stringify(response));
  }
}
```

## RPC Handler Organization

### Domain-Based Handlers

**Location:** `packages/server-core/src/handlers/rpc/`

```
handlers/rpc/
├── auth.ts           # Authentication
├── sessions.ts      # Session CRUD
├── settings.ts      # User settings
├── llm-connections.ts  # AI provider config
├── sources.ts       # Source management
├── automations.ts   # Automation rules
└── ...
```

### Handler Pattern

```typescript
// packages/server-core/src/handlers/rpc/sessions.ts
import type { WebSocket } from 'bun';

export interface SessionHandler {
  list(ws: WebSocket, params: any): Promise<void>;
  create(ws: WebSocket, params: any): Promise<void>;
  get(ws: WebSocket, params: { id: string }): Promise<void>;
  delete(ws: WebSocket, params: { id: string }): Promise<void>;
}

export class SessionHandler implements SessionHandler {
  async list(ws: WebSocket, _params: any) {
    const sessions = await this.sessionService.list();
    this.send(ws, { id: crypto.randomUUID(), result: sessions });
  }

  async create(ws: WebSocket, params: { title?: string }) {
    const session = await this.sessionService.create(params);
    this.send(ws, { id: crypto.randomUUID(), result: session });
  }

  private send(ws: WebSocket, data: any) {
    ws.send(JSON.stringify(data));
  }
}
```

## Session Management

### Session Service

```typescript
// packages/server-core/src/sessions/session-service.ts
export class SessionService {
  private sessions: Map<string, Session> = new Map();

  async create(params: CreateSessionParams): Promise<Session> {
    const session: Session = {
      id: randomUUID(),
      title: params.title || 'Untitled Session',
      createdAt: Date.now(),
      updatedAt: Date.now(),
      messages: [],
    };
    this.sessions.set(session.id, session);
    return session;
  }

  async list(): Promise<SessionMeta[]> {
    return Array.from(this.sessions.values()).map((s) => ({
      id: s.id,
      title: s.title,
      createdAt: s.createdAt,
      updatedAt: s.updatedAt,
    }));
  }

  async get(id: string): Promise<Session | null> {
    return this.sessions.get(id) || null;
  }

  async addMessage(sessionId: string, message: Message): Promise<void> {
    const session = this.sessions.get(sessionId);
    if (session) {
      session.messages.push(message);
      session.updatedAt = Date.now();
    }
  }
}
```

## WebSocket Transport

### Secure WebSocket

```typescript
// With TLS support
const server = Bun.serve({
  port: 8080,
  tls: {
    key: Bun.file('./key.pem'),
    cert: Bun.file('./cert.pem'),
  },
  // ...
});
```

### Connection Management

```typescript
interface ClientConnection {
  id: string;
  ws: WebSocket;
  userId?: string;
  connectedAt: number;
}

export class ConnectionManager {
  private connections: Map<string, ClientConnection> = new Map();

  add(ws: WebSocket): string {
    const id = randomUUID();
    this.connections.set(id, { id, ws, connectedAt: Date.now() });
    return id;
  }

  remove(id: string) {
    this.connections.delete(id);
  }

  broadcast(message: any, exclude?: string[]) {
    const data = JSON.stringify(message);
    for (const [id, conn] of this.connections) {
      if (!exclude?.includes(id)) {
        conn.ws.send(data);
      }
    }
  }
}
```

## WebUI Handler

### Serving Static Files

```typescript
import { fileURLToPath } from 'url';
import { join } from 'path';

const __dirname = fileURLToPath(new URL('.', import.meta.url));

export class WebUIHandler {
  handleRequest(req: Request): Response {
    const url = new URL(req.url);

    if (url.pathname === '/' || url.pathname.startsWith('/assets/')) {
      const filePath = join(__dirname, '../webui/dist', url.pathname);
      return new Response(Bun.file(filePath));
    }

    return new Response(Bun.file(join(__dirname, '../webui/dist/index.html')));
  }
}
```

## Environment Configuration

```bash
# .env
CRAFT_SERVER_TOKEN=your-secret-token
CRAFT_RPC_HOST=localhost
CRAFT_RPC_PORT=8080
CRAFT_HEALTH_PORT=8081
CRAFT_WEBUI_DIR=./webui/dist
```

## Error Handling

```typescript
async function handleRequest(req: Request): Promise<Response> {
  try {
    // Process request
    return await processRequest(req);
  } catch (error) {
    console.error('Request failed:', error);
    return Response.json(
      { error: { code: 500, message: 'Internal server error' } },
      { status: 500 }
    );
  }
}
```

## Health Check

```typescript
// packages/server/src/health.ts
export function createHealthHandler() {
  return async (req: Request): Promise<Response> => {
    const health = {
      status: 'ok',
      timestamp: Date.now(),
      uptime: process.uptime(),
      memory: process.memoryUsage(),
    };
    return Response.json(health);
  };
}

// Register in main server
this.server = Bun.serve({
  port: this.config.port,
  fetch: (req, server) => {
    const url = new URL(req.url);
    if (url.pathname === '/health') {
      return createHealthHandler()(req);
    }
    // ...
  },
});
```

## Docker Integration

**Location:** `Dockerfile.server`

```dockerfile
FROM oven/bun:1-debian
WORKDIR /app
COPY package.json bun.lockb ./
RUN bun install --frozen-lockfile
COPY . .
RUN bun build --target=bun --outdir=dist
EXPOSE 8080 8081
CMD ["bun", "run", "src/index.ts"]
```

## Key Differences from Node.js

| Feature | Bun | Node.js |
|---------|-----|---------|
| HTTP server | Native `Bun.serve()` | `http.createServer()` |
| WebSocket | Native WebSocket in `Bun.serve()` | `ws` library |
| File I/O | `Bun.file()` | `fs.readFile()` |
| SQLite | Native `Bun.sqlite()` | `better-sqlite3` |
| Performance | 2-3x faster | Standard |

## Logging

```typescript
const logger = {
  info: (msg: string, data?: any) => {
    console.log(`[INFO] ${new Date().toISOString()} ${msg}`, data || '');
  },
  error: (msg: string, error?: any) => {
    console.error(`[ERROR] ${new Date().toISOString()} ${msg}`, error || '');
  },
  debug: (msg: string, data?: any) => {
    if (process.env.DEBUG) {
      console.log(`[DEBUG] ${new Date().toISOString()} ${msg}`, data || '');
    }
  },
};
```
