---
title: MDXCodeBlock Playground Development Guide
type: reference
permalink: projects/blogg/guides/codeblock-playground
entity_type: reference
status: evergreen
tags:
- project/blogg
- domain/guides
- domain/mdx
---

# MDXCodeBlock Playground Development Guide

## Goal

Build a component playground system within MDX that can:
1. Reflect into a component (extract source code)
2. Display syntax-highlighted TypeScript/JSX
3. Render live interactive component preview
4. Coexist with `<Pre>` for simple code blocks

## Chosen Approach: React Live + MDX Integration

React Live (Formidable Labs) was selected over Storybook MDX and pure MDX compilation.

**Why:**
- Live editing (~15KB gzipped)
- Actual React component rendering
- Customizable styling and composition
- Active maintenance

**Integration Strategy:**
1. React Live for playground functionality
2. MDX component substitution for integration
3. Existing `CodeBlock` for syntax highlighting
4. `Pre` remains for simple markdown code blocks

## Component Features

### Action Panel
- **Reload**: Remounts target component to reset state (RotateCcw icon, rotating animation)
- **Copy**: Copies source to clipboard (Copy/Check icon, 2s green confirmation)
- **Expand**: Toggles fullscreen view (Maximize2/Minimize2 icon)

### View Modes
- Source only
- Playground only
- Split view (source + live preview)

## Key Files

- `components/new/MDXCodeBlock.tsx` — Main component
- `components/new/README.md` — Feature documentation

## Relations

- child_of [[Next.js Blog (blogg)]]
- explains [[MDXCodeBlock Playground Architecture]]
- uses [[React Live]]