# UI Components Reference

## Overview

This document covers shared React components in `packages/ui/src/components/` and common patterns.

## Component Categories

### Chat Components (`packages/ui/src/components/chat/`)

| Component | Purpose | Key Props |
|-----------|---------|-----------|
| `SessionViewer` | Read-only session transcript | `sessions`, `activeSessionId` |
| `TurnCard` | Email-like display for assistant turns | `turn`, `status` |
| `TurnCardActionsMenu` | Copy/Markdown actions | `turnId`, `content` |
| `ResponseCard` | AI response wrapper | `content`, `activities` |
| `UserMessageBubble` | User message display | `message`, `timestamp` |
| `InlineExecution` | Tool execution progress | `events`, `status` |

### Markdown Components (`packages/ui/src/components/markdown/`)

| Component | Purpose |
|-----------|---------|
| `Markdown` | Full markdown renderer |
| `CodeBlock` | Syntax-highlighted code |
| `TiptapMarkdownEditor` | Rich text editor |
| `MarkdownDatatableBlock` | Data table rendering |
| `MarkdownDiffBlock` | Diff viewer integration |
| `MarkdownMermaidBlock` | Mermaid diagram renderer |
| `MarkdownPdfBlock` | PDF preview |
| `MarkdownSpreadsheetBlock` | Spreadsheet preview |

### UI Primitives (`packages/ui/src/components/ui/`)

| Component | Variants | Purpose |
|----------|----------|---------|
| `Button` | default, destructive, outline, secondary, ghost, link | Actions |
| `Input` | default | Text input |
| `Dialog` | default | Modal dialog |
| `Popover` | default | Floating content |
| `DropdownMenu` | default | Context menu |
| `Tooltip` | default | Hover tooltip |
| `Tabs` | default | Tab navigation |
| `Badge` | default | Labels |
| `Avatar` | default | User image |
| `Spinner` | grid | Loading indicator |
| `Island` | - | Annotation overlay |

### Overlay Components (`packages/ui/src/components/overlay/`)

| Component | Purpose |
|-----------|---------|
| `CodePreviewOverlay` | Code file viewer |
| `TerminalPreviewOverlay` | Terminal output |
| `ImagePreviewOverlay` | Image viewer |
| `PDFPreviewOverlay` | PDF viewer |
| `JSONPreviewOverlay` | JSON viewer |
| `DataTableOverlay` | CSV/table viewer |
| `DocumentFormattedMarkdownOverlay` | Markdown preview |
| `ActivityCardsOverlay` | Activity timeline |

## Common Patterns

### CVA Button Variants

```typescript
// packages/ui/src/components/ui/button.tsx
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva(
  'inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-ring disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        default: 'bg-accent text-background hover:bg-accent/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input bg-background hover:bg-accent hover:text-accent-foreground',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'text-accent underline-offset-4 hover:underline',
      },
      size: {
        default: 'h-9 px-4 py-2',
        sm: 'h-8 px-3 text-xs',
        lg: 'h-10 px-6',
        icon: 'h-8 w-8',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
);
```

### cn() Utility

```typescript
// packages/ui/src/lib/utils.ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

### Platform Abstraction

```typescript
// PlatformProvider context
interface PlatformActions {
  openExternalUrl: (url: string) => Promise<void>;
  copyToClipboard: (text: string) => Promise<void>;
  // ... other platform-specific actions
}

// Using in component
const { copyToClipboard } = usePlatform();
```

### Shiki Code Viewer

```typescript
import { ShikiCodeViewer } from '@craft-agent/ui';

// Highlighting bundled themes
<ShikiCodeViewer
  code={code}
  language="typescript"
  theme="github-dark"
  showLineNumbers
/>
```

### Tiptap Editor

```typescript
import { TiptapMarkdownEditor } from '@craft-agent/ui';

<TiptapMarkdownEditor
  content={content}
  onChange={(html) => setContent(html)}
  placeholder="Enter your markdown..."
  extensions={[StarterKit, Placeholder, Mention]}
/>
```

## Export Structure

```typescript
// packages/ui/src/index.ts
// Context
export { PlatformProvider, usePlatform } from './context';
export { ShikiThemeProvider, useShikiTheme } from './context';

// Chat
export { SessionViewer, TurnCard, UserMessageBubble } from './components/chat';

// Markdown
export { Markdown, TiptapMarkdownEditor } from './components/markdown';

// UI primitives
export { Button, Spinner, Tooltip } from './components/ui';
```

## Styling Guidelines

### Using Design Tokens

```tsx
// Good: use design tokens
<div className="bg-background text-foreground">
  <button className="bg-accent text-accent-foreground hover:bg-accent/90">
    Action
  </button>
</div>

// Avoid: hardcoded colors
<div style={{ backgroundColor: '#faf9fb' }}>
  <button style={{ backgroundColor: '#8b5cf6' }}>Action</button>
</div>
```

### Responsive Classes

```tsx
// Mobile-first with container queries
<div className="@container panel">
  <div className="@sm:text-sm @lg:text-base">
    Responsive text
  </div>
</div>
```

### Animation Classes

```tsx
// Use motion library for complex animations
import { motion } from 'motion/react';

<motion.div
  initial={{ opacity: 0, y: 10 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.2 }}
/>

// Tailwind for simple transitions
<div className="transition-colors hover:bg-accent/90">
  Hover effect
</div>
```