# Migration Guide

## Overview

This guide helps extract Craft Agents architecture into a reusable fullstack development template.

## Step-by-Step Process

### Phase 1: Project Setup

#### 1.1 Initialize Monorepo

```bash
# Create root directory
mkdir my-template && cd my-template

# Initialize Bun workspace
bun init -y

# Create workspace structure
mkdir -p apps packages scripts docs
```

#### 1.2 Root package.json

```json
{
  "name": "my-template",
  "version": "1.0.0",
  "private": true,
  "workspaces": ["apps/*", "packages/*"],
  "scripts": {
    "dev": "bun run --filter=\"electron\" dev",
    "build": "bun run build",
    "typecheck": "bun run typecheck",
    "lint": "bun run lint"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "bun-types": "^1.0.0"
  }
}
```

#### 1.3 TypeScript Configuration

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "jsx": "react-jsx",
    "paths": {
      "@/*": ["./src/*"],
      "@my-template/core": ["../../packages/core/src"],
      "@my-template/shared": ["../../packages/shared/src"],
      "@my-template/ui": ["../../packages/ui/src"]
    }
  }
}
```

### Phase 2: Core Package

#### 2.1 Create core package

```bash
mkdir -p packages/core/src/{types,utils}
```

#### 2.2 core/package.json

```json
{
  "name": "@my-template/core",
  "version": "1.0.0",
  "main": "./src/index.ts",
  "types": "./src/index.ts"
}
```

#### 2.3 core/src/index.ts

```typescript
// Types only - no business logic
export * from './types';
export * from './utils';
```

### Phase 3: Shared Package

#### 3.1 Create shared package

```bash
mkdir -p packages/shared/src/{config,credentials,agent,sessions}
```

#### 3.2 shared/package.json

```json
{
  "name": "@my-template/shared",
  "version": "1.0.0",
  "dependencies": {
    "@my-template/core": "workspace:*",
    "zod": "^3.22.0"
  }
}
```

#### 3.3 Config Pattern

```typescript
// packages/shared/src/config/index.ts
import { z } from 'zod';

const ConfigSchema = z.object({
  app: z.object({
    name: z.string(),
    version: z.string(),
  }),
  server: z.object({
    port: z.number().default(8080),
    host: z.string().default('localhost'),
  }),
  auth: z.object({
    providers: z.array(z.string()),
  }),
});

export type Config = z.infer<typeof ConfigSchema>;

export function loadConfig(): Config {
  // Load from JSON file or environment
}
```

### Phase 4: UI Package

#### 4.1 Create UI package

```bash
mkdir -p packages/ui/src/{components/{ui,chat,markdown},styles,lib}
```

#### 4.2 UI package.json

```json
{
  "name": "@my-template/ui",
  "version": "1.0.0",
  "main": "./src/index.ts",
  "dependencies": {
    "react": "^18.3.0",
    "@radix-ui/react-dialog": "^1.0.0",
    "@radix-ui/react-dropdown-menu": "^1.0.0",
    "lucide-react": "^0.300.0",
    "clsx": "^2.0.0",
    "tailwind-merge": "^2.0.0",
    "class-variance-authority": "^0.7.0"
  }
}
```

#### 4.3 Button Component

```typescript
// packages/ui/src/components/ui/button.tsx
import { forwardRef, type ButtonHTMLAttributes } from 'react';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '../../lib/utils';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md text-sm font-medium',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input bg-background hover:bg-accent hover:text-accent-foreground',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'text-primary underline-offset-4 hover:underline',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 rounded-md px-3',
        lg: 'h-11 rounded-md px-8',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
);

export interface ButtonProps
  extends ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, ...props }, ref) => {
    return (
      <button
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  }
);
```

### Phase 5: Electron App

#### 5.1 Create Electron app

```bash
mkdir -p apps/electron/src/{main,preload,renderer/{components,pages,atoms,hooks}}
```

#### 5.2 Electron package.json

```json
{
  "name": "@my-template/electron",
  "version": "1.0.0",
  "main": "./dist/main/index.js",
  "scripts": {
    "dev": "vite",
    "build": "vite build && electron-builder",
    "typecheck": "tsc --noEmit"
  },
  "dependencies": {
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "jotai": "^2.6.0",
    "@my-template/ui": "workspace:*",
    "@my-template/shared": "workspace:*"
  },
  "devDependencies": {
    "electron": "^39.0.0",
    "electron-builder": "^24.0.0",
    "@vitejs/plugin-react": "^4.0.0",
    "vite": "^6.0.0"
  }
}
```

#### 5.3 Main Process

```typescript
// apps/electron/src/main/index.ts
import { app, BrowserWindow } from 'electron';
import path from 'path';

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

  if (process.env.NODE_ENV === 'development') {
    mainWindow.loadURL('http://localhost:5173');
  } else {
    mainWindow.loadFile(path.join(__dirname, '../renderer/index.html'));
  }
}

app.whenReady().then(createWindow);
```

### Phase 6: Server

#### 6.1 Create server package

```bash
mkdir -p packages/server/src
```

#### 6.2 Server package.json

```json
{
  "name": "@my-template/server",
  "version": "1.0.0",
  "main": "./src/index.ts",
  "scripts": {
    "dev": "bun run src/index.ts",
    "start": "bun run src/index.ts"
  },
  "dependencies": {
    "@my-template/shared": "workspace:*"
  }
}
```

#### 6.3 Bun HTTP Server

```typescript
// packages/server/src/index.ts
const server = Bun.serve({
  port: parseInt(process.env.PORT || '8080'),
  fetch(req, server) {
    const url = new URL(req.url);

    if (url.pathname === '/health') {
      return Response.json({ status: 'ok' });
    }

    // WebSocket upgrade
    if (server.upgrade(req)) {
      return;
    }

    return Response.json({ error: 'Not found' }, { status: 404 });
  },
  websocket: {
    open(ws) {
      console.log('Client connected');
    },
    message(ws, msg) {
      console.log('Received:', msg);
    },
    close(ws, code, reason) {
      console.log('Client disconnected');
    },
  },
});

console.log(`Server running on ${server.port}`);
```

## Best Practices

### 1. Workspace Dependencies

```json
// Always use workspace:* for internal packages
{
  "dependencies": {
    "@my-template/shared": "workspace:*"
  }
}
```

### 2. TypeScript Strict Mode

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true
  }
}
```

### 3. Import Order

```typescript
// 1. React and hooks
// 2. External libraries
// 3. Internal packages
// 4. Local modules
```

### 4. Error Handling

```typescript
// Always use specific error types
try {
  const result = await operation();
} catch (error) {
  if (error instanceof NotFoundError) {
    // Handle 404
  } else if (error instanceof ValidationError) {
    // Handle validation
  } else {
    // Re-throw unknown errors
    throw error;
  }
}
```

### 5. Logging

```typescript
const logger = {
  info: (msg: string, data?: object) => console.log(`[INFO] ${msg}`, data),
  error: (msg: string, error?: Error) => console.error(`[ERROR] ${msg}`, error),
  debug: (msg: string, data?: object) => {
    if (process.env.DEBUG) console.log(`[DEBUG] ${msg}`, data);
  },
};
```

## File Structure Summary

```
my-template/
├── apps/
│   └── electron/
│       ├── src/
│       │   ├── main/
│       │   ├── preload/
│       │   └── renderer/
│       ├── package.json
│       ├── vite.config.ts
│       └── electron-builder.yml
├── packages/
│   ├── core/
│   │   ├── src/
│   │   └── package.json
│   ├── shared/
│   │   ├── src/
│   │   └── package.json
│   ├── ui/
│   │   ├── src/
│   │   └── package.json
│   └── server/
│       ├── src/
│       └── package.json
├── package.json
├── tsconfig.json
└── bunfig.toml
```

## Common Issues

### Issue: Circular Dependencies

**Solution:** Use `core` for types only, never import `shared` from `core`.

### Issue: TypeScript Path Resolution

**Solution:** Use workspace protocol in package.json, not tsconfig paths for runtime.

### Issue: Electron Build Failures

**Solution:** Ensure `node_modules` is excluded from asar archives.
