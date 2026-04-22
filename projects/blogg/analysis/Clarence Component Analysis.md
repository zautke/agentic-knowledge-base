---
title: Clarence Component Analysis
type: reference
permalink: projects/blogg/analysis/clarence-components
entity_type: reference
status: evergreen
tags:
- project/blogg
- domain/analysis
- domain/design-system
---

# Clarence Component Analysis

Deep dive analysis of Theodorus Clarence's website (theodorusclarence.com) design patterns, adapted into a reusable React component library.

## Design Patterns Identified

### 1. Hero Section with Fade Animation
- `fade-in-start` class for entrance animations
- `overflow-hidden` parent for clean transitions
- Progressive disclosure animation

### 2. Gradient Badge
- `bg-gradient-to-r` with custom green colors (`#0F3324`)
- Opacity gradients: 25% to 0%
- Negative margin overlapping design
- Inline flex for text/icon combos

### 3. Blog Introduction Section
- Responsive spacing (`py-12 pt-20 md:py-20`)
- Center alignment focal point
- Flex column vertical stacking

### 4. Advanced Image Handling
- Blur-to-sharp loading transitions (BlurImage component)
- Responsive aspect ratios

### 5. Animation Philosophy
- Subtle micro-interactions only
- Hardware-accelerated transforms
- Desktop-only rotation effects

## Components Created (20+)

### New (5)
- **StackedCards** — 3D card arrangement with CSS transforms
- **GlassIcon** — Glass morphism with gradient borders
- **GradientBadge** — Sophisticated gradient backgrounds
- **AnimatedSection** — Scroll-triggered animations
- **BlogIntro** — Complete blog introduction sections

### Enhanced (3)
- **Button** — 5 variants, 3 sizes, loading/disabled
- **Card** — Multiple variants (hover lift, gradient, minimal, outlined)
- **Badge** — Semantic color system

## Design System Extensions
- **Clarence Green**: `#0F3324` signature color
- **Neutral Scale**: 950, 900 for dark themes
- Group hover parent-child interaction patterns

## Location
`clarence-components/` directory with full TypeScript, exports, and Storybook stories.

## Relations

- child_of [[Next.js Blog (blogg)]]
- informs [[Storybook Component Manual]]
- related_to [[Material UI Design System]]
