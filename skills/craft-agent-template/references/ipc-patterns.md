# IPC Patterns

## Overview

Electron IPC (Inter-Process Communication) connects the main process (Node.js) with the renderer process (React). Craft Agents uses secure, structured IPC patterns.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  Main Process (Node.js)                                     │
│  ├── Window management                                       │
│  ├── IPC handlers (browser.ts, workspace.ts, etc.)        │
│  └── Native APIs                                            │
└─────────────────────────────────────────────────────────────┘
                           │ IPC
┌─────────────────────────────────────────────────────────────┐
│  Preload (Secure Bridge)                                    │
│  └── contextBridge.exposeInMainWorld('electronAPI', {...}) │
└─────────────────────────────────────────────────────────────┘
                           │ contextBridge
┌─────────────────────────────────────────────────────────────┐
│  Renderer Process (React)                                   │
│  └── window.electronAPI.*                                   │
└─────────────────────────────────────────────────────────────┘
```

## Preload Pattern

```typescript
// apps/electron/src/preload/bootstrap.ts
import { contextBridge, ipcRenderer } from 'electron';

// Expose protected methods that allow the renderer process to use
// ipcRenderer without exposing the entire object
contextBridge.exposeInMainWorld('electronAPI', {
  // Browser management
  browser: {
    createPane: (options) => ipcRenderer.invoke('browser:create-pane', options),
    closePane: (paneId) => ipcRenderer.invoke('browser:close-pane', paneId),
    getPane: (paneId) => ipcRenderer.invoke('browser:get-pane', paneId),
  },

  // Window management
  window: {
    minimize: () => ipcRenderer.send('window:minimize'),
    maximize: () => ipcRenderer.send('window:maximize'),
    close: () => ipcRenderer.send('window:close'),
    isMaximized: () => ipcRenderer.invoke('window:is-maximized'),
  },

  // Workspace
  workspace: {
    openDirectory: () => ipcRenderer.invoke('workspace:open-directory'),
    getRecent: () => ipcRenderer.invoke('workspace:get-recent'),
  },

  // Settings
  settings: {
    get: (key) => ipcRenderer.invoke('settings:get', key),
    set: (key, value) => ipcRenderer.invoke('settings:set', key, value),
    getAll: () => ipcRenderer.invoke('settings:get-all'),
  },
});
```

## Main Process Handler

```typescript
// apps/electron/src/main/handlers/browser.ts
import { ipcMain, BrowserWindow } from 'electron';

export function registerBrowserHandlers() {
  ipcMain.handle('browser:create-pane', async (event, options) => {
    const webContents = event.sender;
    const win = BrowserWindow.fromWebContents(webContents);

    // Create new browser pane
    const paneId = await windowManager.createPane(win, options);
    return paneId;
  });

  ipcMain.handle('browser:close-pane', async (event, paneId) => {
    return windowManager.closePane(paneId);
  });

  ipcMain.handle('browser:get-pane', async (event, paneId) => {
    return windowManager.getPane(paneId);
  });
}
```

## Renderer Usage

```typescript
// In React component
const handleOpenDirectory = async () => {
  try {
    const result = await window.electronAPI.workspace.openDirectory();
    if (result) {
      console.log('Opened directory:', result.path);
    }
  } catch (error) {
    console.error('Failed to open directory:', error);
  }
};
```

## Type Safety

### Define API Types

```typescript
// packages/core/src/types/electron-api.ts
export interface ElectronAPI {
  browser: {
    createPane: (options: PaneOptions) => Promise<string>;
    closePane: (paneId: string) => Promise<void>;
    getPane: (paneId: string) => Promise<Pane | null>;
  };
  window: {
    minimize: () => void;
    maximize: () => void;
    close: () => void;
    isMaximized: () => Promise<boolean>;
  };
  workspace: {
    openDirectory: () => Promise<DirectoryResult | null>;
    getRecent: () => Promise<string[]>;
  };
  settings: {
    get: <T>(key: string) => Promise<T | undefined>;
    set: <T>(key: string, value: T) => Promise<void>;
    getAll: () => Promise<Record<string, unknown>>;
  };
}

declare global {
  interface Window {
    electronAPI: ElectronAPI;
  }
}
```

## IPC Channels

### Channel Naming Convention

```
{domain}:{action}
```

| Domain | Purpose | Example |
|--------|---------|---------|
| `browser` | Browser pane management | `browser:create-pane` |
| `window` | Window operations | `window:minimize` |
| `workspace` | Workspace/directory | `workspace:open-directory` |
| `settings` | App settings | `settings:get` |
| `session` | Session management | `session:create` |
| `auth` | Authentication | `auth:login` |

## Security

### Context Isolation

```typescript
// main/index.ts
const mainWindow = new BrowserWindow({
  webPreferences: {
    preload: path.join(__dirname, 'preload.js'),
    contextIsolation: true,     // Enabled - isolate preload from renderer
    nodeIntegration: false,      // Disabled - no Node.js in renderer
    sandbox: true,               // Enabled - sandbox renderer
  },
});
```

### Input Validation

```typescript
// Always validate IPC inputs
ipcMain.handle('settings:set', async (event, key, value) => {
  // Validate key format
  if (typeof key !== 'string' || !/^[\w.-]+$/.test(key)) {
    throw new Error('Invalid settings key');
  }

  // Validate value type
  if (value !== undefined && typeof value !== 'string' && typeof value !== 'number' && typeof value !== 'boolean') {
    throw new Error('Invalid settings value type');
  }

  // Proceed with validated input
  return settingsStore.set(key, value);
});
```

## Deep Link Pattern

```typescript
// apps/electron/src/main/deep-link.ts
export function registerDeepLinks() {
  // Register scheme (called once)
  if (process.defaultApp) {
    if (process.argv.length >= 2) {
      app.setAsDefaultProtocolClient('craft-agent', process.execPath, [
        path.resolve(process.argv[1]),
      ]);
    }
  } else {
    app.setAsDefaultProtocolClient('craft-agent');
  }
}

// Handle incoming links
app.on('open-url', (event, url) => {
  event.preventDefault();
  handleDeepLink(url);
});
```

## Window State Persistence

```typescript
// apps/electron/src/main/window-state.ts
import { app } from 'electron';
import * as fs from 'fs';

interface WindowState {
  width: number;
  height: number;
  x?: number;
  y?: number;
  isMaximized: boolean;
}

export function loadWindowState(): WindowState {
  const statePath = path.join(app.getPath('userData'), 'window-state.json');

  try {
    if (fs.existsSync(statePath)) {
      return JSON.parse(fs.readFileSync(statePath, 'utf-8'));
    }
  } catch (error) {
    console.error('Failed to load window state:', error);
  }

  // Default state
  return {
    width: 1200,
    height: 800,
    isMaximized: false,
  };
}

export function saveWindowState(state: WindowState): void {
  const statePath = path.join(app.getPath('userData'), 'window-state.json');

  try {
    fs.writeFileSync(statePath, JSON.stringify(state));
  } catch (error) {
    console.error('Failed to save window state:', error);
  }
}
```