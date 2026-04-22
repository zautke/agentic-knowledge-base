---
title: design-system-workflow
type: note
permalink: operations/design-system-workflow
tags:
- operation
- draft
- workflow
- design-system
- development
---

# Design System Development Workflow

**Status**: draft
**Last Updated**: 2026-01-01

## Daily Development Flow

### 1. Adding a New Component

```bash
# Create component file
touch packages/ui/src/primitives/dialog.tsx

# Add to tsup entry points
# Edit packages/ui/tsup.config.ts

# Add export to package.json exports map
# Edit packages/ui/package.json

# Build and test
pnpm -r build
pnpm -r test
```

### 2. Modifying Design Tokens

```bash
# Edit token values
code packages/design-tokens/src/css/tokens.css

# Rebuild tokens package
cd packages/design-tokens && pnpm build

# Test in consuming app
```

### 3. Publishing Updates

```bash
# Create changeset
pnpm changeset

# Commit changes
git add .
git commit -m "feat(ui): add Dialog component"

# Push to main (triggers CI/CD)
git push origin main
```

---

## Component Checklist

When creating new components:

- [ ] Use CVA for variants
- [ ] Include forwardRef for ref forwarding
- [ ] Add displayName for DevTools
- [ ] Use cn() for class merging
- [ ] Export types alongside component
- [ ] Add to tsup entry points
- [ ] Add to package.json exports
- [ ] Write tests with Testing Library

---

## Debugging

### Token not applying?
1. Check CSS import order (tokens → theme → animations)
2. Verify @theme mapping exists for the token
3. Inspect computed styles in DevTools

### Component not tree-shaking?
1. Verify separate entry point in tsup.config.ts
2. Check package.json exports map
3. Ensure sideEffects: false in package.json

---

## Relations
- implements [[design-system-deployment-protocol]]
- follows [[shadcn-tailwind-primer]]