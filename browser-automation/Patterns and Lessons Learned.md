---
title: Patterns and Lessons Learned
type: note
permalink: browser-automation/patterns-and-lessons-learned
---

# Browser Automation Patterns & Lessons Learned

## Circular Dependencies in Theme Overrides
- **Problem**: In MUI theme development, importing the final theme instance into component overrides that are then used to build that same theme creates a circular dependency. This manifests as `Cannot access 'theme' before initialization` in the browser.
- **Solution**: Use a `themeStub.ts` or a "base theme" that contains only the static parts (palette, typography, spacing, breakpoints) to satisfy the overrides. Alternatively, use function-based style overrides which receive the theme at runtime.

## Evidence-Based Verification
- **Technique**: Use `page.on('console', ...)` and `page.on('pageerror', ...)` in Playwright to capture browser-side errors that might cause the UI to fail silently (e.g., hidden `body`).
- **Lesson**: If the `body` is hidden or elements are missing, check the browser console for TDZ errors or initialization failures.

## MUI v7 Migration Patterns
- **Selector Specificity**: Avoid using state keys like `active` or `selected` directly in `styleOverrides`. Use nested selectors like `&.Mui-active` within the `root` slot to ensure correct specificity and avoid MUI warnings.
- **Naming Conventions**: Ensure CSS properties in JS objects use camelCase (`boxShadow`) rather than kebab-case (`box-shadow`).

## Vite Configuration
- When using monorepo-like structures where assets are in a parent directory, you may need to adjust `server.fs.allow` to prevent 403 Forbidden errors for fonts or other static assets.