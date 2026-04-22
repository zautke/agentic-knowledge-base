---
title: 2026-01-21 - MUI7 Storybook Migration
type: note
permalink: agent-kb/sessions/2026-01-21-mui7-storybook-migration
---

# Session: 2026-01-21 - MUI7 Storybook Migration

## Context
Executed a comprehensive migration of the `blogg` project's design system to Material UI v7 and Storybook 8+. The goal was to modernize dependencies, ensure pixel-perfect fidelity, and establish a "best practice" Storybook environment for future development.

## Observations

- [technique] **Theming Adapter**: Migrated from custom decorators to `@storybook/addon-themes` using `withThemeFromJSXProvider`. This standardizes theme injection (Light/Dark) and `CssBaseline` handling, reducing boilerplate and brittleness.
- [technique] **Font Optimization**: Bundled `Roboto` and `Material Icons` via `@fontsource` packages instead of CDNs. This eliminates FOUT (Flash of Unstyled Text) and enables offline development/testing.
- [technique] **Interaction Testing**: Integrated `storybook-addon-pseudo-states`. This allows visual regression tools to capture hover/focus states deterministically without requiring script execution.
- [fact] **Next.js 16 Compatibility**: Next.js 16 enables Turbopack by default. We encountered conflicts with the project's custom `webpack` config in `next.config.ts`. Adding the `--webpack` flag to the `dev` script was a necessary workaround to maintain custom build logic.
- [pattern] **Composite Stories**: Moving beyond atomic components to "Complex" patterns (e.g., `SmartButton`, `FormDialog`, `SortableTable`) significantly improves documentation value by showing components in realistic state-managed contexts.
- [refactor] **Theme Architecture**: Centralized theme logic into `EnhancedThemeProvider` and `finalEnhancedTheme` exports. This "single source of truth" prevents drift between the Next.js app and the Storybook preview environment.

## Work Completed
1.  **Dependency Upgrade**: Next.js 16.1.4, Storybook 10+, MUI 7.3.7.
2.  **Configuration**: Updated `.storybook/preview.js` and `main.js`.
3.  **Porting**: Generated stories for 27+ core MUI components.
4.  **Enhancement**: Added "Complex" component examples demonstrating state and composition.
5.  **Validation**: Verified E2E tests (47/48 pass) and static Storybook build.

## Relations
- related_to [[domains/Material UI Design System]]
