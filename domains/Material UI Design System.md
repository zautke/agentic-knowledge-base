---
title: Material UI Design System
type: note
entity_type: note
permalink: agent-kb/domains/material-ui-design-system
created: 2026-04-21
modified: 2026-05-28
status: evergreen
tags:
- project/agent-kb
- domain/design-system
- domain/material-ui
- domain/frontend
- status/evergreen
- source/largo
- machine/largo
---

# Material UI Design System (MUI 7 + Storybook)

## Overview
This domain covers the implementation and maintenance of the project's design system, built on Material UI v7, Next.js 16, and Storybook 8+.

## Architecture

### Tech Stack
- **Framework**: Next.js 16 (App Router)
- **UI Library**: Material UI v7
- **Styling**: Emotion (MUI default) + Tailwind CSS v4 (utility classes)
- **Documentation**: Storybook 8+ (Vite builder)

### Theme System
The theme is centralized in `customTheme/index.ts`.
- **Provider**: `EnhancedThemeProvider` wraps `MUIThemeProvider` and handles context.
- **Theme Object**: `finalEnhancedTheme` merges base settings with component overrides.
- **CSS Variables**: Configured for `cssVarPrefix: 'mui'`.

## Storybook Integration

### Configuration
Located in `.storybook/`.
- **Main**: Uses `@storybook/nextjs-vite` framework.
- **Preview**:
    - **Decorators**: Uses `withThemeFromJSXProvider` from `@storybook/addon-themes`.
    - **Fonts**: Imports `@fontsource/roboto` and `@fontsource/material-icons` directly.

### Best Practices
1.  **Theming**: Always use `EnhancedThemeProvider` in decorators to match App behavior.
2.  **Interaction**: Use `storybook-addon-pseudo-states` to document hover/focus styles.
3.  **Composition**: Group complex, stateful patterns under `MUI7/Complex/` (e.g., `SmartButton`, `SortableTable`) to demonstrate real-world usage.
4.  **Automation**: Use `scripts/generate-mui-stories.js` to scaffold new component stories.

## Common Patterns

### Smart Components
For components with internal state (loading, success, error):
```tsx
// Example: SmartButton
const LoadingButton = ({ loading, success, ...props }) => {
  // Internal state logic for transitions
  return <Button startIcon={loading ? <CircularProgress /> : ...} ... />
}
```

### Dialogs and Portals
Dialogs should demonstrate state management in the Story:
```tsx
const FormDialog = () => {
  const [open, setOpen] = useState(false);
  return (
    <>
      <Button onClick={() => setOpen(true)}>Open</Button>
      <Dialog open={open} ... />
    </>
  )
}
```

## Troubleshooting
- **Turbopack Conflict**: If `next dev` fails with Webpack config errors, use `next dev --webpack` to force Webpack mode.
- **Font Issues**: Verify `@fontsource` imports in `.storybook/preview.js` if icons or fonts are missing.

## Relations

- related_to [[design-system-workflow]]
- related_to [[Browser Automation]]
- related_to [[shadcn-tailwind-design-system-founding-document]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-21 | Domain note captured (git import date) | Anchor the MUI 7 + Storybook design-system domain knowledge | KB seeding |
| 2026-05-28 | Completed frontmatter to 7-field KB contract (added entity_type/created/modified/status, added `domain/design-system`, `domain/material-ui`, `domain/frontend`, `project/agent-kb` tags); added Relations section linking the design-system workflow note, the browser-automation domain, and the shadcn/Tailwind founding document | Note was an orphan with source/machine-only tags and no relations | KB curation hygiene sweep (operations/reference partition) |