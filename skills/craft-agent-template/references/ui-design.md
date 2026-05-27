# UI Design System

## Styling Architecture

### Tailwind CSS v4 CSS-First Configuration

Craft Agents uses Tailwind CSS v4's **CSS-first configuration** approach (no tailwind.config.js):

```css
/* apps/electron/src/renderer/index.css */
@import "tailwindcss" source(none);
@plugin "@tailwindcss/typography";

/* Tell Tailwind where to scan for utility classes */
@source "../renderer/App.tsx";
@source "../../../../packages/ui/src/components/**/*.{ts,tsx,jsx}";

/* ANIMATABLE CSS CUSTOM PROPERTIES
   Use @property for smooth theme transitions (Chromium/Electron full support) */
@property --background {
  syntax: "<color>";
  inherits: true;
  initial-value: oklch(0.98 0.003 265);
}

@property --foreground {
  syntax: "<color>";
  inherits: true;
  initial-value: oklch(0.185 0.01 270);
}

/* CRAFT AGENT THEME - 6 Color System

   Base colors:
   - background: Light/dark surface
   - foreground: Text and icons
   - accent: Brand purple (highlights, Auto mode)
   - info: Amber (warnings, Ask mode)
   - success: Green (connected, checkmarks)
   - destructive: Red (errors, failed)

   Variants:
   - /50 = 50% alpha (transparency) - for borders, hovers, overlays
   - -50 = 50% mix toward background (solid) - for Explore mode

   Example: bg-foreground/10, text-accent, bg-foreground-60
*/

/* Light Mode */
:root {
  /* === 6 BASE COLORS === */
  --background: oklch(0.98 0.003 265);
  --background-elevated: color-mix(in srgb, var(--foreground) 1.5%, var(--background));
  --foreground: oklch(0.185 0.01 270);
  --foreground-dimmed: color-mix(in srgb, var(--foreground) 80%, var(--background));
  --foreground-rgb: 38, 36, 42;
  --accent: oklch(0.62 0.13 293);
  --accent-rgb: 104, 78, 133;
  --info: oklch(0.75 0.16 70);
  --info-rgb: 180, 120, 40;
  --success: oklch(0.55 0.17 145);
  --success-rgb: 34, 120, 60;
  --destructive: oklch(0.58 0.24 28);
  --destructive-rgb: 180, 60, 50;

  /* === SHADOW OPACITY (light mode = subtle) === */
  --shadow-border-opacity: 0.08;
  --shadow-blur-opacity: 0.06;

  /* === SHADOW VALUES === */
  --shadow-minimal:
    rgba(0, 0, 0, 0) 0px 0px 0px 0px,
    rgba(0, 0, 0, 0) 0px 0px 0px 0px,
    rgba(var(--foreground-rgb), 0.06) 0px 0px 0px 1px,
    rgba(0, 0, 0, var(--shadow-blur-opacity)) 0px 1px 1px -0.5px,
    rgba(0, 0, 0, var(--shadow-blur-opacity)) 0px 3px 3px -1.5px;

  /* === MIX VARIANTS (solid interpolation toward background) === */
  --foreground-1\.5: color-mix(in srgb, var(--foreground) 1.5%, var(--background));
  --foreground-2: color-mix(in srgb, var(--foreground) 2%, var(--background));
  --foreground-3: color-mix(in srgb, var(--foreground) 3%, var(--background));
  --foreground-5: color-mix(in srgb, var(--foreground) 5%, var(--background));
  --foreground-10: color-mix(in srgb, var(--foreground) 10%, var(--background));
  --foreground-20: color-mix(in srgb, var(--foreground) 20%, var(--background));
  --foreground-50: color-mix(in srgb, var(--foreground) 50%, var(--background));
  --foreground-90: color-mix(in srgb, var(--foreground) 90%, var(--background));
  --foreground-95: color-mix(in srgb, var(--foreground) 95%, var(--background));
}

/* Dark Mode */
.dark {
  --background: oklch(0.145 0.015 270);
  --background-elevated: color-mix(in srgb, var(--foreground) 1.5%, var(--background));
  --foreground: oklch(0.95 0.01 270);
  --foreground-dimmed: color-mix(in srgb, var(--foreground) 80%, var(--background));
  --foreground-rgb: 237, 236, 240;
  --accent: oklch(0.65 0.22 293);
  --accent-rgb: 130, 75, 185;
  --info: oklch(0.78 0.14 70);
  --info-rgb: 210, 155, 55;
  --success: oklch(0.60 0.17 145);
  --success-rgb: 50, 140, 80;
  --destructive: oklch(0.65 0.22 28);
  --destructive-rgb: 200, 70, 60;

  /* === SHADOW OPACITY (stronger in dark mode) === */
  --shadow-border-opacity: 0.15;
  --shadow-blur-opacity: 0.12;
}

/* Theme integration via @theme inline */
@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-accent: var(--accent);
  --color-info: var(--info);
  --color-success: var(--success);
  --color-destructive: var(--destructive);

  /* Z-Index Scale */
  --z-index-base: 0;
  --z-index-local: 10;
  --z-index-sticky: 20;
  --z-index-titlebar: 40;
  --z-index-panel: 50;
  --z-index-dropdown: 100;
  --z-index-tooltip: 150;
  --z-index-modal: 200;
  --z-index-overlay: 300;
  --z-index-fullscreen: 350;
  --z-index-floating-backdrop: 390;
  --z-index-floating-menu: 400;
  --z-index-island-overlay: 390;
  --z-index-island: 400;
  --z-index-island-popover: 410;
  --z-index-splash: 600;

  /* Custom breakpoints */
  --container-3xs: 16rem;
  --container-panel-compact: 28rem;
  --container-panel-medium: 40rem;
  --container-mobile: 48rem;

  --font-sans: var(--font-sans);
  --font-mono: var(--font-mono);
}
```

**Key differences from v3:**

| Feature | v3 | v4 |
|---------|----|----|
| Config location | tailwind.config.js | CSS @theme |
| Dark mode | class strategy | .dark class on html |
| Color format | HEX/RGB | OKLCH preferred |
| JIT | Always on | CSS-first |

## Color System

### 6-Color Theme with OKLCH

Craft Agents uses **perceptually uniform OKLCH colors** for consistent rendering:

| Token | Light Mode | Dark Mode | Usage |
|-------|------------|-----------|-------|
| `background` | `oklch(0.98 0.003 265)` | `oklch(0.145 0.015 270)` | Page surface |
| `foreground` | `oklch(0.185 0.01 270)` | `oklch(0.95 0.01 270)` | Text/icons |
| `accent` | `oklch(0.62 0.13 293)` | `oklch(0.65 0.22 293)` | Brand purple (Auto) |
| `info` | `oklch(0.75 0.16 70)` | `oklch(0.78 0.14 70)` | Amber (Ask, warnings) |
| `success` | `oklch(0.55 0.17 145)` | `oklch(0.60 0.17 145)` | Green (connected) |
| `destructive` | `oklch(0.58 0.24 28)` | `oklch(0.65 0.22 28)` | Red (errors) |

### OKLCH Benefits

- **Perceptually uniform** - human eye perceives color changes evenly
- **Gamut clamping** - accurate color reproduction across displays
- **Easy dark mode** - adjust lightness channel only
- **HDR ready** - supports wider color gamut

## Z-Index Scale

Centralized z-index registry:

```css
--z-base: 0;              /* Default content */
--z-local: 10;           /* Local stacking (sticky headers) */
--z-sticky: 20;          /* Sticky section headers */
--z-titlebar: 40;         /* Window titlebar drag region */
--z-panel: 50;           /* App panels (sidebar, chat, detail) */
--z-dropdown: 100;       /* Dropdowns, popovers, selects */
--z-tooltip: 150;        /* Tooltips (above dropdowns) */
--z-modal: 200;          /* Dialogs, drawers */
--z-overlay: 300;        /* Full-screen overlays */
--z-fullscreen: 350;     /* Fullscreen content viewers */
--z-island: 400;         /* Annotation island menu surface */
--z-island-popover: 410; /* Popovers from island content */
--z-splash: 600;         /* Splash screen */
```

## Typography

### Font Stack

```css
--font-sans: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
--font-mono: "JetBrains Mono", ui-monospace, SFMono-Regular, "SF Mono", Menlo, Consolas, monospace;
--font-size-base: 15px;
```

### Scale

| Element | Size | Weight |
|---------|------|--------|
| Body | 15px | 400 |
| Small | 13px | 400 |
| H1 | 24px | 600 |
| H2 | 20px | 600 |
| H3 | 16px | 600 |

## Component Library

### Core Primitives (`packages/ui/src/components/ui/`)

| Component | Variants | Purpose |
|----------|----------|---------|
| `Button` | default, destructive, outline, secondary, ghost, link | Actions |
| `Input` | default | Text input |
| `Dialog` | default | Modal dialog |
| `Drawer` | default | Slide-out panel |
| `Popover` | default | Floating content |
| `Tooltip` | default | Hover tooltip |
| `Tabs` | default | Tab navigation |
| `DropdownMenu` | default | Context menu |
| `Badge` | default | Labels |
| `Avatar` | default | User image |
| `Spinner` | grid | Loading indicator |
| `Island` | - | Annotation overlay |

### Button Variants (CVA)

```typescript
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva('inline-flex items-center justify-center', {
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
});
```

### cn() Utility

```typescript
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

## Dark Mode

### Implementation

```typescript
// Toggle via .dark class on html element
document.documentElement.classList.toggle('dark');

// Theme provider pattern
export function useTheme() {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = () => {
    setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));
    document.documentElement.classList.toggle('dark');
  };

  return { theme, setTheme, toggleTheme };
}
```

### CSS Variables

```css
/* Light mode (default) */
:root {
  --background: oklch(0.98 0.003 265);
  --foreground: oklch(0.185 0.01 270);
}

/* Dark mode */
.dark {
  --background: oklch(0.145 0.015 270);
  --foreground: oklch(0.95 0.01 270);
}
```

## Shadow System

```css
/* Minimal shadow - just border ring */
.shadow-minimal {
  box-shadow:
    rgba(0, 0, 0, 0) 0px 0px 0px 0px,
    rgba(0, 0, 0, 0) 0px 0px 0px 0px,
    rgba(var(--foreground-rgb), 0.06) 0px 0px 0px 1px;
}

/* Medium shadow for panels */
.shadow-middle {
  box-shadow:
    rgba(0, 0, 0, 0) 0px 0px 0px 0px,
    rgba(0, 0, 0, 0) 0px 0px 0px 0px,
    rgba(var(--foreground-rgb), 0.06) 0px 0px 0px 1px,
    rgba(0, 0, 0, var(--shadow-blur-opacity)) 0px 1px 1px -0.5px,
    rgba(0, 0, 0, var(--shadow-blur-opacity)) 0px 3px 3px -1.5px,
    rgba(0, 0, 0, var(--shadow-blur-opacity)) 0px 6px 6px -3px;
}

/* Tinted shadow - use with style={{ '--shadow-color': 'var(--info-rgb)' }} */
.shadow-tinted {
  box-shadow:
    rgba(var(--shadow-color), calc(var(--shadow-border-opacity) * 1.5)) 0px 0px 0px 1px,
    rgba(var(--shadow-color), var(--shadow-border-opacity)) 0px 1px 1px -0.5px,
    rgba(var(--shadow-color), var(--shadow-blur-opacity)) 0px 3px 3px -1.5px,
    rgba(var(--shadow-color), calc(var(--shadow-blur-opacity) * 0.67)) 0px 6px 6px -3px;
}
```

## Animation Patterns

### Loading Spinner (Grid)

```css
.spinner {
  display: inline-grid;
  grid-template-columns: repeat(3, 1fr);
  width: 1em;
  height: 1em;
  gap: 0.08em;
}

.spinner-cube {
  background-color: currentColor;
  animation: spinner-grid 1.3s infinite ease-in-out;
  transform: scale3d(0.5, 0.5, 1);
}

@keyframes spinner-grid {
  0%, 70%, 100% { transform: scale3d(0.5, 0.5, 1); }
  35% { transform: scale3d(0, 0, 1); }
}
```

### Shimmer Effect (Optimistic UI)

```css
.animate-shimmer::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(
    90deg,
    transparent 0%,
    oklch(from var(--foreground) l c h / 0.06) 50%,
    transparent 100%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s ease-in-out infinite;
}
```

### Scrollbar Styling

```css
/* Hide scrollbar until hover */
.scrollbar-hover::-webkit-scrollbar {
  width: 4px;
}
.scrollbar-hover::-webkit-scrollbar-thumb {
  background: transparent;
}
.group:hover .scrollbar-hover::-webkit-scrollbar-thumb {
  background: var(--border);
}
```

## Toast Notifications (Sonner)

```css
[data-sonner-toast] {
  box-shadow: var(--shadow-modal-small) !important;
  width: 356px !important;
}

/* State-based tinted backgrounds */
[data-sonner-toast][data-type="error"] {
  background: linear-gradient(rgba(var(--destructive-rgb), 0.15), rgba(var(--destructive-rgb), 0.15)),
    oklch(from var(--popover) l c h / 0.8) !important;
}

[data-sonner-toast][data-type="success"] {
  background: linear-gradient(rgba(var(--success-rgb), 0.15), rgba(var(--success-rgb), 0.15)),
    oklch(from var(--popover) l c h / 0.8) !important;
}
```

## Layout Constants

### Chat Layout

```typescript
// packages/ui/src/lib/layout.ts
export const CHAT_LAYOUT = {
  maxWidth: 'max-w-[840px]',
  containerPadding: 'px-5 py-8',
  messageSpacing: 'space-y-2.5',
};

export const OVERLAY_LAYOUT = {
  modalBreakpoint: 99999,
  modalMaxWidth: 1100,
  modalMaxHeightPercent: 85,
};
```

## Responsive Design

### Container Queries

```css
/* Panel-constrained layouts */
@container panel (max-width: 448px) {
  [data-slot="dropdown-menu-content"] {
    max-width: calc(100cqw - 24px);
  }

  /* Stack settings rows vertically */
  [data-layout="settings-row"] {
    flex-direction: column;
    align-items: stretch;
  }
}

/* Shell container for touch scaling (1.3x) */
@container shell (max-width: 768px) {
  .panel-header-btn { padding: 8px; }
  .send-btn { width: 36px; height: 36px; }
}
```

## Glassmorphism (Scenic Mode)

```css
/* Full-window background with glass UI panels */
html[data-scenic]::before {
  background-image: var(--background-image);
  background-size: cover;
  z-index: -2;
}

html[data-scenic] .shadow-middle {
  backdrop-filter: blur(8px);
  background: oklch(0.2 0.005 270 / 0.55);
}
```

## Accessibility

### Focus Management

```css
/* Focus visible for keyboard navigation */
:focus-visible {
  outline: none;
  --tw-ring-offset-width: var(--ring-offset);
  --tw-ring-width: var(--ring-width);
  --tw-ring-color: var(--ring);
}

/* Custom focus ring */
.focus-ring:focus-visible {
  box-shadow: 0 0 0 2px var(--background), 0 0 0 4px var(--accent);
}
```

### ARIA Patterns

```tsx
// Icon button with label
<button aria-label="Close dialog">
  <XIcon className="h-4 w-4" />
</button>

// Hidden label for screen readers
<button aria-label={label}>
  <Icon className="h-4 w-4" />
</button>
```

## Icon System

Uses Lucide React for consistent icons:

```tsx
import { X, Plus, Settings, Search } from 'lucide-react';

<Button>
  <Plus className="h-4 w-4 mr-2" />
  Add Item
</Button>
```

## Rich Text Editor (Tiptap)

```tsx
import { useEditor } from '@tiptap/react';
import StarterKit from '@tiptap/starter-kit';

const editor = useEditor({
  extensions: [StarterKit],
  content: '<p>Hello World</p>',
});

return (
  <div className="prose">
    <EditorContent editor={editor} />
  </div>
);
```

### Extensions Used

- `StarterKit` - Basic formatting
- `Placeholder` - Empty state placeholder
- `Mention` - @mentions
- `SlashCommands` - /command menu