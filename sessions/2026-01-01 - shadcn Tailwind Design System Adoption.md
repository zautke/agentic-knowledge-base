---
title: shadcn-tailwind-primer
type: note
permalink: primers/shadcn-tailwind-primer
tags:
- primer
- draft
- design-system
- shadcn
- tailwind-css
---

# shadcn + Tailwind CSS 4.1 Design System Primer

**Status**: draft
**Last Updated**: 2026-01-01

## Overview

This primer covers building production design systems using shadcn/ui component patterns, Tailwind CSS 4.1's CSS-native theming, and npm package distribution.

---

## Core Concepts

### shadcn/ui Philosophy
- [fact] Copy-paste distribution model, not traditional npm package
- [fact] Built on Radix UI primitives for accessibility
- [fact] Uses CVA (class-variance-authority) for type-safe variants
- [technique] Components are owned by you, not a vendor

### Tailwind CSS 4.x Architecture
- [fact] CSS-native configuration via `@theme inline` directive
- [fact] No JavaScript config files needed
- [fact] 5x faster full builds, 100x faster incremental
- [fact] Requires Safari 16.4+, Chrome 111+, Firefox 128+

### Design Token Layers
```
Base Palette (raw colors)
    ↓
Semantic Tokens (purpose-driven: --primary, --background)
    ↓
Component Variants (CVA definitions)
```

---

## Essential Patterns

### 1. CSS Variable Theming
```css
:root {
  --primary: oklab(0.6087 0.1068 0.0867);
  --background: oklch(0.99 0.002 85);
}

.dark {
  --primary: oklab(0.6064 0.1091 0.0853);
  --background: oklch(0.16 0.01 85);
}

@theme inline {
  --color-primary: var(--primary);
  --color-background: var(--background);
}
```

### 2. CVA Component Pattern
```tsx
const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground',
        outline: 'border border-input bg-background',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 px-3',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
)
```

### 3. cn() Utility
```ts
import { clsx } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

---

## Package Structure

```
packages/
├── design-tokens/
│   ├── src/css/
│   │   ├── tokens.css      # Base palette + semantics
│   │   ├── theme.css       # @theme inline mapping
│   │   └── animations.css  # Transitions + keyframes
│   └── package.json
└── ui/
    ├── src/
    │   ├── primitives/     # Button, Card, Input, etc.
    │   ├── components/     # Composed components
    │   └── utils/cn.ts
    └── package.json
```

---

## Key Dependencies

| Package | Purpose |
|---------|---------|
| `tailwindcss@^4.0` | CSS framework |
| `class-variance-authority` | Type-safe variants |
| `clsx` | Conditional classes |
| `tailwind-merge` | Class deduplication |
| `@radix-ui/*` | Accessible primitives |

---

## Relations
- implements [[shadcn-tailwind-design-system-founding-document]]
- guides [[design-system-deployment-protocol]]