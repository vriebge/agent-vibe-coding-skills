# Electron Architecture

## Overview

Craft Agents uses Electron for cross-platform desktop deployment with a strict main/renderer separation.

## Process Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     MAIN PROCESS                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Window    │  │     IPC     │  │   Native    │         │
│  │  Manager    │  │   Handler   │  │     API     │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
                            │ IPC (contextBridge)
┌─────────────────────────────────────────────────────────────┐
│                   RENDERER PROCESS                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │    React    │  │    Jotai    │  │     Vite    │         │
│  │     App     │  │    State    │  │     HMR     │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

## Main Process (Node.js)

**Location:** `apps/electron/src/main/`

**Responsibilities:**
- Window creation and lifecycle management
- Native API access (file system, dialogs, etc.)
- IPC handler registration
- Subprocess management (MCP servers, Pi agent)
- Auto-updater setup

### Main Entry (`index.ts`)

```typescript
import { app, BrowserWindow } from 'electron';
import { registerIpcHandlers } from './ipc';

let mainWindow: BrowserWindow | null = null;

function createWindow() {
  mainWindow = new BrowserWindow({
    width: 1200,
    height: 800,
    minWidth: 800,
    minHeight: 600,
    webPreferences: {
      preload: path.join(__dirname, '../preload/index.js'),
      contextIsolation: true,
      nodeIntegration: false,
    },
  });

  // Load content
  if (process.env.NODE_ENV === 'development') {
    mainWindow.loadURL('http://localhost:5173');
    mainWindow.webContents.openDevTools();
  } else {
    mainWindow.loadFile(path.join(__dirname, '../renderer/index.html'));
  }
}

app.whenReady().then(() => {
  registerIpcHandlers();
  createWindow();
});
```

### IPC Handler Registration

**Location:** `apps/electron/src/main/ipc.ts`

```typescript
import { ipcMain } from 'electron';
import { sessionsHandlers } from './handlers/sessions';
import { settingsHandlers } from './handlers/settings';

export function registerIpcHandlers() {
  sessionsHandlers(ipcMain);
  settingsHandlers(ipcMain);
  // Add more handlers as needed
}
```

### IPC Handler Pattern

**Location:** `apps/electron/src/main/handlers/`

```typescript
// Example: sessions handler
export function sessionsHandlers(ipcMain: IpcMain) {
  ipcMain.handle('sessions:list', async () => {
    // Return list of sessions
  });

  ipcMain.handle('sessions:create', async (_, data) => {
    // Create new session
  });

  ipcMain.handle('sessions:delete', async (_, sessionId) => {
    // Delete session
  });
}
```

## Preload Script

**Location:** `apps/electron/src/preload/`

**Purpose:** Secure bridge between main and renderer processes.

### Preload Entry

```typescript
import { contextBridge, ipcRenderer } from 'electron';

// Expose protected methods to renderer
contextBridge.exposeInMainWorld('electronAPI', {
  // Invoke pattern (request/response)
  invoke: (channel: string, ...args: any[]) => {
    // Whitelist channels for security
    const validChannels = ['sessions:list', 'sessions:create', 'sessions:delete'];
    if (validChannels.includes(channel)) {
      return ipcRenderer.invoke(channel, ...args);
    }
  },

  // On pattern (one-way events)
  on: (channel: string, callback: (...args: any[]) => void) => {
    const subscription = (_event: IpcRendererEvent, ...args: any[]) => callback(...args);
    ipcRenderer.on(channel, subscription);
    return () => ipcRenderer.removeListener(channel, subscription);
  },

  // Send pattern (fire and forget)
  send: (channel: string, ...args: any[]) => {
    ipcRenderer.send(channel, ...args);
  },
});
```

### Type-Safe API Exposure

```typescript
// In renderer, declare the API type:
declare global {
  interface Window {
    electronAPI: {
      invoke: (channel: string, ...args: any[]) => Promise<any>;
      on: (channel: string, callback: (...args: any[]) => void) => () => void;
      send: (channel: string, ...args: any[]) => void;
    };
  }
}
```

## Renderer Process (React + Vite)

**Location:** `apps/electron/src/renderer/`

**Responsibilities:**
- React UI rendering
- State management with Jotai
- User interaction handling
- IPC communication via preload API

### Renderer Entry

```typescript
// main.tsx
import React from 'react';
import { createRoot } from 'react-dom/client';
import { Provider } from 'jotai';
import { App } from './App';
import './styles/index.css';

const container = document.getElementById('root');
if (container) {
  const root = createRoot(container);
  root.render(
    <React.StrictMode>
      <Provider>
        <App />
      </Provider>
    </React.StrictMode>
  );
}
```

### AppShell Layout

The main layout uses `AppShell` component with resizable panels:

```typescript
// App.tsx
import { AppShell } from './components/app-shell/AppShell';

export function App() {
  return (
    <AppShell
      leftSidebar={<SessionList />}
      mainContent={<ChatDisplay />}
      rightSidebar={<SourcesPanel />}
    />
  );
}
```

## Vite Configuration

**Location:** `apps/electron/vite.config.ts`

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  root: path.join(__dirname, 'src/renderer'),
  base: './',
  build: {
    outDir: path.join(__dirname, 'dist'),
    emptyOutDir: true,
  },
  server: {
    port: 5173,
  },
});
```

## Electron Builder Configuration

**Location:** `apps/electron/electron-builder.yml`

```yaml
appId: com.craft.agent
productName: Craft Agent
copyright: Copyright (c) 2024 Craft Docs

directories:
  output: dist
  buildResources: build

files:
  - dist/**/*
  - package.json

extraResources:
  - from: node_modules/@anthropic-ai/claude-agent-sdk
    to: node_modules/@anthropic-ai/claude-agent-sdk

mac:
  category: public.app-category.developer-tools
  hardenedRuntime: true
  gatekeeperAssess: false

win:
  target:
    - nsis

linux:
  target:
    - AppImage
```

## IPC Channel Conventions

| Channel | Direction | Purpose |
|---------|-----------|---------|
| `sessions:list` | invoke | Get all sessions |
| `sessions:create` | invoke | Create new session |
| `sessions:delete` | invoke | Delete session |
| `sessions:stream` | send/on | Stream messages |
| `settings:get` | invoke | Get settings |
| `settings:set` | invoke | Update settings |

## Security Best Practices

1. **Context Isolation:** Always enabled
2. **Node Integration:** Disabled in renderer
3. **Preload Script:** Use contextBridge, not nodeIntegration
4. **Channel Whitelist:** Validate IPC channels in preload
5. **CSP Headers:** Configure Content Security Policy

## Common Patterns

### Opening Dialog from Renderer

```typescript
// In preload
contextBridge.exposeInMainWorld('electronAPI', {
  openFileDialog: () => ipcRenderer.invoke('dialog:openFile'),
});

// In main handler
ipcMain.handle('dialog:openFile', async () => {
  const result = await dialog.showOpenDialog(mainWindow, {
    properties: ['openFile'],
  });
  return result;
});
```

### File System Access

```typescript
// In main handler
ipcMain.handle('fs:readFile', async (_, filePath: string) => {
  return await fs.readFile(filePath, 'utf-8');
});
```

### Window Management

```typescript
// In main
ipcMain.handle('window:minimize', () => {
  mainWindow?.minimize();
});

ipcMain.handle('window:maximize', () => {
  if (mainWindow?.isMaximized()) {
    mainWindow.unmaximize();
  } else {
    mainWindow?.maximize();
  }
});
```
