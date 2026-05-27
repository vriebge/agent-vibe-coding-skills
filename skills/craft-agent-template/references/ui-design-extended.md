# UI Design Extended - Color System & Tokens

## Overview

Craft Agents uses a sophisticated **OKLCH-based color system** with progressive opacity scales, tinted shadows, and semantic color tokens. This document provides detailed reference for the theming implementation.

## Color Architecture

### Base 6 Colors

| Token | Light Mode | Dark Mode | Usage |
|-------|------------|-----------|-------|
| `background` | `oklch(0.98 0.003 265)` | `oklch(0.145 0.015 270)` | Page surface |
| `foreground` | `oklch(0.185 0.01 270)` | `oklch(0.95 0.01 270)` | Text/icons |
| `accent` | `oklch(0.62 0.13 293)` | `oklch(0.65 0.22 293)` | Brand purple (Auto mode) |
| `info` | `oklch(0.75 0.16 70)` | `oklch(0.78 0.14 70)` | Amber (Ask mode, warnings) |
| `success` | `oklch(0.55 0.17 145)` | `oklch(0.60 0.17 145)` | Green (connected) |
| `destructive` | `oklch(0.58 0.24 28)` | `oklch(0.65 0.22 28)` | Red (errors, failed) |

### Derived Colors

```css
/* Elevated surface (panels, cards) */
--background-elevated: color-mix(in srgb, var(--foreground) 1.5%, var(--background));

/* Dimmed text for unfocused panels */
--foreground-dimmed: color-mix(in srgb, var(--foreground) 80%, var(--background));

/* User message bubble */
--user-message-bubble: oklch(from var(--foreground) l c h / 0.05);
--user-message-bubble-dimmed: oklch(from var(--foreground) l c h / 0.03);
```

## RGB Variants for Shadows

RGB variants enable colored shadows without alpha issues:

```css
/* Light mode */
--foreground-rgb: 38, 36, 42;
--accent-rgb: 104, 78, 133;
--info-rgb: 180, 120, 40;
--success-rgb: 34, 120, 60;
--destructive-rgb: 180, 60, 50;

/* Dark mode (brighter) */
.dark {
  --foreground-rgb: 237, 236, 240;
  --accent-rgb: 130, 75, 185;
  --info-rgb: 210, 155, 55;
  --success-rgb: 50, 140, 80;
  --destructive-rgb: 200, 70, 60;
}
```

## Shadow Opacity Scale

| Mode | `--shadow-border-opacity` | `--shadow-blur-opacity` |
|------|---------------------------|-------------------------|
| Light | `0.08` | `0.06` |
| Dark | `0.15` | `0.12` |

## Predefined Shadow Values

```css
/* Minimal - just border ring */
--shadow-minimal:
  rgba(0, 0, 0, 0) 0px 0px 0px 0px,
  rgba(0, 0, 0, 0) 0px 0px 0px 0px,
  rgba(var(--foreground-rgb), 0.06) 0px 0px 0px 1px;

/* Minimal flat (no blur) */
--shadow-minimal-flat:
  rgba(0, 0, 0, 0) 0px 0px 0px 0px,
  rgba(0, 0, 0, 0) 0px 0px 0px 0px,
  rgba(var(--foreground-rgb), 0.06) 0px 0px 0px 1px;

/* Medium shadow for panels */
--shadow-middle:
  rgba(0, 0, 0, 0) 0px 0px 0px 0px,
  rgba(0, 0, 0, 0) 0px 0px 0px 0px,
  rgba(var(--foreground-rgb), 0.06) 0px 0px 0px 1px,
  rgba(0, 0, 0, var(--shadow-blur-opacity)) 0px 1px 1px -0.5px,
  rgba(0, 0, 0, var(--shadow-blur-opacity)) 0px 3px 3px -1.5px,
  rgba(0, 0, 0, var(--shadow-blur-opacity)) 0px 6px 6px -3px;

/* Strong shadow for modals */
--shadow-modal-small:
  rgba(0, 0, 0, 0) 0px 0px 0px 0px,
  rgba(0, 0, 0, 0) 0px 0px 0px 0px,
  rgba(var(--foreground-rgb), 0.06) 0px 0px 0px 1px,
  rgba(0, 0, 0, calc(var(--shadow-blur-opacity) * 0.67)) 0px 1px 1px -0.5px,
  rgba(0, 0, 0, calc(var(--shadow-blur-opacity) * 0.67)) 0px 3px 3px 0px,
  rgba(0, 0, 0, calc(var(--shadow-blur-opacity) * 0.33)) 0px 6px 6px 0px,
  rgba(0, 0, 0, calc(var(--shadow-blur-opacity) * 0.33)) 0px 12px 12px 0px,
  rgba(0, 0, 0, calc(var(--shadow-blur-opacity) * 0.33)) 0px 24px 24px 0px;
```

## Tinted Shadow Pattern

Colored shadows using CSS variable `--shadow-color`:

```css
.shadow-tinted {
  --shadow-color: 0, 0, 0;
  box-shadow:
    rgba(var(--shadow-color), calc(var(--shadow-border-opacity) * 1.5)) 0px 0px 0px 1px,
    rgba(var(--shadow-color), var(--shadow-border-opacity)) 0px 1px 1px -0.5px,
    rgba(var(--shadow-color), var(--shadow-blur-opacity)) 0px 3px 3px -1.5px,
    rgba(var(--shadow-color), calc(var(--shadow-blur-opacity) * 0.67)) 0px 6px 6px -3px;
}

/* Dark mode: outline only, no blur glow */
.dark .shadow-tinted {
  box-shadow:
    oklch(from rgb(var(--shadow-color)) 0.425 c h / 0.6) 0px 0px 0px 1px;
}
```

**Usage:**
```tsx
<div style={{ '--shadow-color': 'var(--info-rgb)' }} className="shadow-tinted">
  Info-tinted shadow
</div>
```

## Foreground Mix Variants

Solid interpolation toward background (for separators, Explore mode):

```css
/* Solid colors (e.g., separators) */
--foreground-1\.5: color-mix(in srgb, var(--foreground) 1.5%, var(--background));
--foreground-2: color-mix(in srgb, var(--foreground) 2%, var(--background));
--foreground-3: color-mix(in srgb, var(--foreground) 3%, var(--background));
--foreground-5: color-mix(in srgb, var(--foreground) 5%, var(--background));
--foreground-10: color-mix(in srgb, var(--foreground) 10%, var(--background));
--foreground-20: color-mix(in srgb, var(--foreground) 20%, var(--background));
--foreground-50: color-mix(in srgb, var(--foreground) 50%, var(--background));
--foreground-90: color-mix(in srgb, var(--foreground) 90%, var(--background));
--foreground-95: color-mix(in srgb, var(--foreground) 95%, var(--background));
```

## Semantic Text Colors

For tinted text using oklab (perceptually uniform):

```css
/* Using oklab for perceptually uniform mixing without hue shifts */
--success-text: color-mix(in oklab, var(--success) 50%, var(--foreground));
--destructive-text: color-mix(in oklab, var(--destructive) 50%, var(--foreground));
--info-text: color-mix(in oklab, var(--info) 50%, var(--foreground));
```

## ShadCN Compatibility

Derived tokens for shadcn/ui component compatibility:

```css
--secondary: oklch(from var(--foreground) l c h / 0.05);
--secondary-foreground: var(--foreground);
--muted: oklch(from var(--foreground) l c h / 0.05);
--muted-foreground: var(--foreground-50);
--card: var(--background);
--card-foreground: var(--foreground);
--popover: var(--background);
--popover-foreground: var(--foreground);
--border: oklch(from var(--foreground) l c h / 0.05);
--input: oklch(from var(--foreground) l c h / 0.1);
--ring: oklch(from var(--foreground) l c h / 0.25);
```

## Theme JSON Schema

Themes are defined in `apps/electron/resources/themes/*.json`:

```json
{
  "name": "Default",
  "description": "Clean purple-tinted neutral theme",
  "author": "Craft Agent",
  "license": "MIT",
  "supportedModes": ["light", "dark"],
  "shikiTheme": {
    "light": "github-light",
    "dark": "github-dark"
  },
  "background": "#faf9fb",
  "foreground": "#1a1625",
  "accent": "#8b5cf6",
  "info": "#d97706",
  "success": "#16a34a",
  "destructive": "#dc2626",
  "dark": {
    "background": "#1e1d21",
    "foreground": "#f5f5f7",
    "accent": "#a78bfa",
    "info": "#fbbf24",
    "success": "#22c55e",
    "destructive": "#ef4444"
  }
}
```

### Supported Theme Modes

| Mode | Description |
|------|-------------|
| `light` | Light mode only |
| `dark` | Dark mode only |
| `light, dark` | Both modes supported |

## Scenic Mode (Glassmorphism)

Full-window background with glass panels:

```css
/* Background image layer */
html[data-scenic]::before {
  background-image: var(--background-image);
  background-size: cover;
  background-position: center;
  z-index: -2;
}

/* Dark overlay for readability */
html.dark[data-scenic]::after {
  background: linear-gradient(to right, rgba(0, 0, 0, 0.4), rgba(0, 0, 0, 0.4));
  z-index: -1;
}

/* Glass panel effect */
html[data-scenic] .shadow-middle {
  backdrop-filter: blur(8px);
  background: oklch(0.2 0.005 270 / 0.55);
}
```

### Activating Scenic Mode

```typescript
// Set theme with backgroundImage
document.documentElement.dataset.scenic = 'true';
document.documentElement.style.setProperty('--background-image', 'url(/scenic-bg.jpg)');
```