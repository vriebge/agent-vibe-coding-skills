# React Patterns

## State Management with Jotai

### Core Concepts

Jotai uses atoms (pieces of state) and reactions (computed state):

```typescript
import { atom } from 'jotai';

// Primitive atom
const countAtom = atom(0);

// Derived atom
const doubleCountAtom = atom((get) => get(countAtom) * 2);
```

### Session Isolation Pattern

Each session has isolated state using atomFamily:

```typescript
// apps/electron/src/renderer/atoms/sessions.ts
import { atom } from 'jotai';
import { atomFamily } from 'jotai/utils';

// Session-specific state
const sessionAtomFamily = atomFamily((sessionId: string) => {
  return atom({
    messages: [],
    status: 'idle' as 'idle' | 'loading' | 'streaming' | 'error',
    metadata: null as SessionMeta | null,
  });
});

// Usage in component
function ChatDisplay({ sessionId }: { sessionId: string }) {
  const [sessionState, setSessionState] = useAtom(sessionAtomFamily(sessionId));
  // ...
}
```

### Key Atoms

| Atom | Purpose |
|------|---------|
| `sessionAtomFamily(sessionId)` | Per-session state |
| `activeSessionIdAtom` | Currently viewed session |
| `sessionMetaMapAtom` | Lightweight metadata for list |
| `loadedSessionsAtom` | Lazy loading tracking |

### Why atomFamily?

**Without atomFamily:**
```typescript
// All sessions in one atom - causes re-renders
const sessionsAtom = atom<Map<string, SessionState>>(new Map());

function SessionList() {
  const [sessions] = useAtom(sessionsAtom);
  // Any session update causes re-render
}
```

**With atomFamily:**
```typescript
// Each session isolated - only relevant session re-renders
function SessionList() {
  const [session] = useAtom(sessionAtomFamily(sessionId));
  // Only this session's updates trigger re-render
}
```

## Component Architecture

### AppShell Layout

**Location:** `apps/electron/src/renderer/components/app-shell/`

**Components:**
- `AppShell.tsx` - Main layout container
- `TopBar.tsx` - Application header
- `LeftSidebar.tsx` - Navigation + session list
- `MainContentPanel.tsx` - Chat/messages area
- `RightSidebar.tsx` - Optional panels

### Resizable Panels

Uses `react-resizable-panels`:

```typescript
import { Panel, PanelGroup, PanelResizeHandle } from 'react-resizable-panels';

function AppShell() {
  return (
    <PanelGroup direction="horizontal">
      <Panel defaultSize={20} minSize={15} maxSize={40}>
        <LeftSidebar />
      </Panel>
      <PanelResizeHandle />
      <Panel defaultSize={60} minSize={30}>
        <MainContentPanel />
      </Panel>
      <PanelResizeHandle />
      <Panel defaultSize={20} minSize={15} maxSize={40}>
        <RightSidebar />
      </Panel>
    </PanelGroup>
  );
}
```

### Component Organization

```
components/
├── app-shell/       # Main layout
│   ├── AppShell.tsx
│   ├── TopBar.tsx
│   ├── LeftSidebar.tsx
│   └── ...
├── ui/             # Electron-specific overrides
├── browser/       # Browser components
├── settings/       # Settings panels
└── onboarding/     # Onboarding flow
```

## Custom Hooks

### useSession Hook

```typescript
// hooks/useSession.ts
export function useSession(sessionId: string) {
  const [sessionState, setSessionState] = useAtom(sessionAtomFamily(sessionId));
  const [activeSessionId] = useAtom(activeSessionIdAtom);

  const isActive = activeSessionId === sessionId;

  const sendMessage = useCallback(async (message: string) => {
    setSessionState((prev) => ({ ...prev, status: 'loading' }));
    // Send via IPC or WebSocket
  }, [sessionId]);

  return {
    sessionState,
    sendMessage,
    isActive,
  };
}
```

### useFocusZone Hook

Keyboard navigation focus management:

```typescript
interface FocusZoneOptions {
  direction: 'vertical' | 'horizontal';
  wrap?: boolean;
}

export function useFocusZone(options: FocusZoneOptions) {
  // Focus management logic
}
```

## Component Patterns

### Composition Pattern

```typescript
// Reusable wrapper
function Card({ children, className }: { children: ReactNode; className?: string }) {
  return (
    <div className={cn('rounded-lg border bg-card', className)}>
      {children}
    </div>
  );
}

// Specialized card
function SessionCard({ session }: { session: SessionMeta }) {
  return (
    <Card className="hover:bg-accent/10 cursor-pointer">
      <div className="p-4">{session.title}</div>
    </Card>
  );
}
```

### Controlled vs Uncontrolled

**Controlled (recommended for complex state):**
```typescript
function Input({ value, onChange }: { value: string; onChange: (v: string) => void }) {
  return <input value={value} onChange={(e) => onChange(e.target.value)} />;
}
```

**Uncontrolled (for simple local state):**
```typescript
function SearchInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  const handleSearch = () => {
    console.log(inputRef.current?.value);
  };

  return <input ref={inputRef} />;
}
```

## Performance Patterns

### Memoization

```typescript
// Heavy computation
const sortedMessagesAtom = atom((get) => {
  const messages = get(messagesAtom);
  return [...messages].sort((a, b) => a.timestamp - b.timestamp);
});

// Component memoization
const SessionCard = React.memo(function SessionCard({ session }: { session: SessionMeta }) {
  return <div>{session.title}</div>;
});
```

### Lazy Loading

```typescript
// Messages loaded on-demand
const messagesAtom = atomFamily((sessionId: string) => {
  return atom(async (get) => {
    const meta = get(sessionMetaMapAtom)[sessionId];
    if (!meta?.loaded) {
      // Fetch messages from server
      return await fetchMessages(sessionId);
    }
    return get(cachedMessagesAtomFamily(sessionId));
  });
});
```

## Context Usage

### When to Use Context

- Theme (dark/light mode)
- User settings
- Global modals

### When to Use Jotai

- Session state
- Form state
- Async data
- Derived state

## Error Handling

```typescript
function ChatDisplay({ sessionId }: { sessionId: string }) {
  const [sessionState, setSessionState] = useAtom(sessionAtomFamily(sessionId));

  if (sessionState.status === 'error') {
    return <ErrorState message={sessionState.error?.message} onRetry={retry} />;
  }

  return (
    <div>
      {sessionState.messages.map((msg) => (
        <Message key={msg.id} {...msg} />
      ))}
    </div>
  );
}
```

## TypeScript Patterns

```typescript
// Type-safe props
interface ButtonProps {
  variant?: 'default' | 'destructive' | 'outline';
  size?: 'sm' | 'md' | 'lg';
  children: ReactNode;
  onClick?: () => void;
}

// Generic component
function List<T>({ items, renderItem }: { items: T[]; renderItem: (item: T) => ReactNode }) {
  return <div>{items.map(renderItem)}</div>;
}
```
