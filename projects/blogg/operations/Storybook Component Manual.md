---
title: Storybook Component Manual
type: reference
permalink: projects/blogg/operations/storybook-manual
entity_type: reference
status: evergreen
tags:
- project/blogg
- domain/storybook
- domain/components
---

# Storybook Component Manual

## Running

```bash
pnpm run storybook        # Dev server on port 6006
pnpm run build-storybook  # Production build
```

## Organization

Two main categories:
- **Clarence/** — Clarence Design System components
- **Project/** — Existing project components

Each component includes: interactive controls, story variants, usage examples, accessibility info.

## Clarence Design System Components

### UI Components
- **Button**: 5 variants (primary/secondary/outline/ghost/danger), 3 sizes, loading/disabled states
- **Card**: Hover lift, gradient, minimal, outlined variants
- **Badge**: Semantic colors (success/warning/error/info), sizes, pill shape
- **BlurImage**: Progressive blur-to-sharp loading

### Layout Components
- **Container**: 5 responsive width variants
- **Header**: Backdrop blur, mobile menu, theme switch

### Content Components
- **BlogCard**: Image, metadata, tags, hover effects
- **StackedCards**: 3D CSS transforms, staggered z-index
- **GlassIcon**: Glass morphism with gradient borders
- **GradientBadge**: Gradient backgrounds, opacity transitions
- **AnimatedSection**: Scroll-triggered entrance animations

## Development Workflow

1. Create component in appropriate category
2. Add `.stories.tsx` file co-located with component
3. Define variants as separate story exports
4. Use `argTypes` for interactive controls
5. Verify in Storybook before committing

## Relations

- child_of [[Next.js Blog (blogg)]]
- documents [[Material UI Design System]]
- uses [[Storybook]]