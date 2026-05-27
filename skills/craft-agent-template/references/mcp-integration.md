# MCP Integration

## Overview

Craft Agents implements the Model Context Protocol (MCP) for standardized tool extensions across AI providers.

## MCP SDK Integration

### Installation

```bash
bun add @modelcontextprotocol/sdk
```

### Core Concepts

| Concept | Description |
|---------|-------------|
| **MCP Server** | Provides tools to AI clients |
| **MCP Client** | Consumes tools from servers |
| **Tool** | A callable function with schema |
| **Resource** | Static data provided to AI |
| **Prompt** | Templated text for AI interactions |

## MCP Server Implementation

### Session MCP Server

**Location:** `packages/session-mcp-server/src/index.ts`

```typescript
import { Server } from '@modelcontextprotocol/sdk/server';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio';

const server = new Server(
  {
    name: 'craft-session-server',
    version: '1.0.0',
  },
  {
    capabilities: {
      tools: {},
      resources: {},
    },
  }
);

// Register tool handlers
server.setRequestHandler('tools/list', async () => {
  return {
    tools: [
      {
        name: 'read_file',
        description: 'Read contents of a file',
        inputSchema: {
          type: 'object',
          properties: {
            path: { type: 'string', description: 'File path to read' },
          },
          required: ['path'],
        },
      },
      {
        name: 'list_directory',
        description: 'List contents of a directory',
        inputSchema: {
          type: 'object',
          properties: {
            path: { type: 'string', description: 'Directory path' },
          },
          required: ['path'],
        },
      },
      // More tools...
    ],
  };
});

server.setRequestHandler('tools/call', async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case 'read_file':
      return await handleReadFile(args);
    case 'list_directory':
      return await handleListDirectory(args);
    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

## Tool Context Pattern

### Abstract Tool Context

**Location:** `packages/session-tools-core/src/context.ts`

```typescript
export interface ToolContext {
  platform: 'electron' | 'server' | 'cli';
  homeDir: string;
  tempDir: string;
  credentials: CredentialStore;
  sandbox: SandboxContext;
}

export interface CredentialStore {
  get(key: string): Promise<string | null>;
  set(key: string, value: string): Promise<void>;
  delete(key: string): Promise<void>;
}

export interface SandboxContext {
  fileSystem: FileSystemAbstraction;
  process: ProcessAbstraction;
}
```

### Tool Handler Pattern

```typescript
// packages/session-tools-core/src/handlers/file-handlers.ts
import type { ToolContext } from '../context';

export const fileHandlers = {
  async readFile(ctx: ToolContext, args: { path: string }): Promise<ToolResult> {
    try {
      const content = await ctx.sandbox.fileSystem.readFile(args.path);
      return {
        content: [
          {
            type: 'text',
            text: content,
          },
        ],
      };
    } catch (error) {
      return {
        content: [
          {
            type: 'text',
            text: `Error reading file: ${error.message}`,
          },
        ],
        isError: true,
      };
    }
  },
};
```

## MCP Client Pool

### HTTP Transport

**Location:** `packages/shared/src/mcp/`

```typescript
import { Client } from '@modelcontextprotocol/sdk/client';

// HTTP MCP Client
export class CraftMcpClient {
  private client: Client;

  constructor(config: { url: string; apiKey?: string }) {
    this.client = new Client(
      {
        name: 'craft-mcp-client',
        version: '1.0.0',
      },
      {
        capabilities: {
          tools: {},
        },
      }
    );
  }

  async connect() {
    await this.client.connect({
      transport: 'http',
      url: this.config.url,
      headers: this.config.apiKey
        ? { Authorization: `Bearer ${this.config.apiKey}` }
        : {},
    });
  }

  async listTools() {
    const response = await this.client.request(
      { method: 'tools/list' },
      { method: 'tools/list', params: {} }
    );
    return response.tools;
  }

  async callTool(name: string, args: any) {
    const response = await this.client.request(
      { method: 'tools/call' },
      {
        method: 'tools/call',
        params: { name, arguments: args },
      }
    );
    return response;
  }
}
```

### Stdio Transport

```typescript
import { spawn } from 'child_process';

export class StdioMcpClient {
  private proc: ReturnType<typeof spawn>;
  private client: Client;

  constructor(config: { command: string; args: string[] }) {
    this.proc = spawn(config.command, config.args, {
      stdio: ['pipe', 'pipe', 'pipe'],
    });

    this.client = new Client(
      { name: 'craft-stdio-client', version: '1.0.0' },
      { capabilities: { tools: {} } }
    );
  }

  async connect() {
    const transport = new StdioServerTransport(
      this.proc.stdin,
      this.proc.stdout,
      this.proc.stderr
    );
    await this.client.connect(transport);
  }
}
```

## MCP Pool Server

Manages multiple MCP connections:

```typescript
export class McpPoolServer {
  private clients: Map<string, CraftMcpClient> = new Map();

  async addClient(name: string, config: McpClientConfig) {
    const client = new CraftMcpClient(config);
    await client.connect();
    this.clients.set(name, client);
  }

  async removeClient(name: string) {
    const client = this.clients.get(name);
    if (client) {
      await client.disconnect();
      this.clients.delete(name);
    }
  }

  async callTool(clientName: string, toolName: string, args: any) {
    const client = this.clients.get(clientName);
    if (!client) {
      throw new Error(`Unknown MCP client: ${clientName}`);
    }
    return await client.callTool(toolName, args);
  }

  async broadcastToolCall(toolName: string, args: any) {
    const results = await Promise.all(
      Array.from(this.clients.entries()).map(([name, client]) =>
        client.callTool(toolName, args).then((r) => ({ name, result: r }))
      )
    );
    return results;
  }
}
```

## Tool Definition Schema

### JSON Schema for Tools

```typescript
const toolSchema = {
  name: 'string',           // Unique identifier
  description: 'string',   // Human-readable description
  inputSchema: {
    type: 'object',
    properties: {
      // Tool-specific parameters
    },
    required: ['param1'],
  },
};

// Example: Search files tool
{
  name: 'search_files',
  description: 'Search for files matching a pattern',
  inputSchema: {
    type: 'object',
    properties: {
      pattern: {
        type: 'string',
        description: 'Glob pattern to match (e.g., "*.ts")',
      },
      directory: {
        type: 'string',
        description: 'Directory to search in',
      },
    },
    required: ['pattern'],
  },
}
```

## Session-Scoped Tools

Tools are scoped to sessions for isolation:

```typescript
export class SessionScopedMcpServer {
  private servers: Map<string, Server> = new Map();

  createForSession(sessionId: string): Server {
    if (this.servers.has(sessionId)) {
      return this.servers.get(sessionId)!;
    }

    const server = new Server(
      { name: `session-${sessionId}`, version: '1.0.0' },
      { capabilities: { tools: {} } }
    );

    // Configure session-specific tools
    this.configureSessionTools(server, sessionId);
    this.servers.set(sessionId, server);
    return server;
  }

  private configureSessionTools(server: Server, sessionId: string) {
    server.setRequestHandler('tools/list', async () => ({
      tools: getSessionTools(sessionId),
    }));

    server.setRequestHandler('tools/call', async (request) => {
      const { name, arguments: args } = request.params;
      // Add session context to all tool calls
      return await handleToolCall(name, { ...args, sessionId });
    });
  }
}
```

## MCP with Pi Agent

Pi agent runs as a subprocess with MCP tools:

```typescript
// packages/pi-agent-server/src/index.ts
import { spawn } from 'child_process';

export class PiAgentServer {
  private proc: ReturnType<typeof spawn>;

  constructor(config: { tools: ToolDefinition[] }) {
    this.proc = spawn('pi-agent', ['--tools', JSON.stringify(config.tools)], {
      stdio: ['pipe', 'pipe', 'pipe'],
    });
  }

  async sendMessage(message: string): Promise<string> {
    return new Promise((resolve, reject) => {
      let response = '';
      this.proc.stdout.on('data', (data) => {
        response += data.toString();
      });
      this.proc.stdout.once('end', () => resolve(response));
      this.proc.stderr.once('data', (data) => reject(data.toString()));
      this.proc.stdin.write(JSON.stringify({ message }));
      this.proc.stdin.end();
    });
  }
}
```

## Error Handling

```typescript
try {
  const result = await client.callTool(toolName, args);
  return result;
} catch (error) {
  if (error.code === 'METHOD_NOT_FOUND') {
    return { content: [{ type: 'text', text: `Tool not found: ${toolName}` }], isError: true };
  }
  if (error.code === 'INVALID_PARAMS') {
    return { content: [{ type: 'text', text: `Invalid arguments: ${error.message}` }], isError: true };
  }
  return { content: [{ type: 'text', text: `Error: ${error.message}` }], isError: true };
}
```

## Best Practices

1. **Session Isolation** - Each session should have its own MCP context
2. **Tool Validation** - Validate tool schemas before registration
3. **Error Boundaries** - Wrap tool calls in try/catch
4. **Logging** - Log all tool calls for debugging
5. **Resource Cleanup** - Close MCP connections when session ends
