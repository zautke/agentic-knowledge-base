---
title: shadcn-tailwind-design-system-founding-document
type: note
permalink: meta/founding-documents/shadcn-tailwind-design-system-founding-document
tags:
- founding-document
- design-system
- shadcn
- tailwind-css
- npm
- github-actions
- theming
---

# Founding Document: shadcn + Tailwind CSS 4.1 Design System with npm Deployment

## Executive Summary

This skill adoption establishes mastery of building production-ready design systems using shadcn/ui patterns, Tailwind CSS 4.1's CSS-native configuration, and automated npm package publishing via GitHub Actions. The existing @braisenly codebase demonstrates advanced practices (OKLab colors, multi-entry bundling) but lacks deployment automation and documentation.

---

## Research Synthesis

### From Web Research

- [fact] **shadcn/ui Philosophy**: Copy-paste component distribution, not traditional npm dependency. Eliminates vendor lock-in, enables AI tools to read/modify code directly.
- [fact] **Tailwind CSS 4.x Revolution**: CSS-first configuration via `@theme` directive. Full builds 5x faster, incremental 100x faster. Requires Safari 16.4+, Chrome 111+.
- [fact] **npm OIDC Transition**: Classic tokens deprecated, revoked November 19, 2025. Must use trusted publishing with short-lived cryptographically-signed tokens.
- [technique] **Semantic Release**: Use `semantic-release` package with Angular commit conventions for automatic versioning, changelogs, and npm publishing.
- [lesson] **CSS Preprocessor Incompatibility**: Tailwind 4.x cannot coexist with Sass, Less, or Stylus.

### From Code Research

- [fact] **Current Architecture**: Monorepo with `@braisenly/design-tokens` (CSS exports, Tailwind preset) + `@braisenly/ui` (React components with CVA).
- [technique] **Multi-entry tsup Pattern**: Separate entry points for each component enables tree-shaking and selective imports.
- [technique] **OKLab Color System**: Perceptually uniform color space ensures accessible contrast across light/dark themes.
- [technique] **cn() Utility Pattern**: `twMerge(clsx(...inputs))` prevents class conflicts while maintaining composability.
- [fact] **"use client" Banners**: All components marked for Next.js App Router compatibility.

### From Knowledge Base

- [fact] **No existing primers** for React, CSS/Tailwind, npm publishing, or GitHub Actions CI/CD.
- [fact] **No GitHub Actions workflows** exist in the repository - must be created from scratch.
- [fact] **Testing infrastructure incomplete** - Vitest configured but no test files.
- [fact] **Documentation missing** - No README, API docs, or usage examples.

---

## Consolidated Observations

### Facts (Verified Truths)
- Tailwind CSS 4.x uses CSS-native `@theme inline` configuration, eliminating JavaScript config files
- shadcn/ui distributes source code, not compiled packages - maintenance responsibility transfers to consuming team
- npm requires scoped packages to use `--access public` flag for public visibility
- Container queries now built into Tailwind 4.x core (`@min-*`, `@max-*` variants)
- The @braisenly packages already use peerDependencies pattern correctly (React 18-19, Tailwind 4.0+)

### Techniques (Actionable Methods)
- **Three-layer Token Architecture**: Base palette → Semantic tokens → Component variants
- **Dark Mode via Class Selector**: `.dark { --primary: ... }` pattern avoids dark: prefix duplication
- **Workspace Protocol**: Use `workspace:*` for internal monorepo dependencies
- **Component Entry Points**: Export individual components for optimal tree-shaking vs barrel exports
- **OIDC Trusted Publishing**: Configure GitHub Actions with `id-token: write` permission for npm

### Principles (Guiding Truths)
- Copy-paste > package dependencies for maximum flexibility
- CSS variables for runtime theming, Tailwind for static utilities
- Semantic tokens abstract visual design from implementation
- AI-readable code enables future tooling opportunities

### Questions (Open for Investigation)
- Should we add Storybook for component documentation?
- What testing strategy for design system components (visual regression)?
- How to handle version bumps across monorepo packages?
- Should animations be opt-in or included by default?
- Changesets vs semantic-release for monorepo versioning?

---

## Recommended Approach

### Mental Model

```
┌─────────────────────────────────────────────────────────┐
│                    Consumer Apps                          │
│   (Next.js, Vite, Remix - import CSS + components)       │
└──────────────────────────┬──────────────────────────────┘
                           │
┌──────────────────────────┴──────────────────────────────┐
│               @braisenly/ui (React Components)           │
│   - Button, Card, Input, Badge, Select, Sonner          │
│   - Uses CVA for variants, Radix for accessibility      │
│   - Imports tokens as peer dependency                    │
└──────────────────────────┬──────────────────────────────┘
                           │
┌──────────────────────────┴──────────────────────────────┐
│           @braisenly/design-tokens (CSS + Preset)        │
│   - tokens.css (base palette + semantic tokens)          │
│   - theme.css (@theme inline mapping)                    │
│   - animations.css (transitions + keyframes)             │
└─────────────────────────────────────────────────────────┘
```

### Critical Success Factors
1. **Complete GitHub Actions workflow** for automated npm publishing with OIDC
2. **Add pnpm-workspace.yaml** and root package.json for proper monorepo management
3. **Test coverage** for all primitives (Vitest + Testing Library)
4. **README documentation** with usage examples for both packages
5. **Versioning strategy** (likely Changesets for monorepo)

### Anti-Patterns to Avoid
- ❌ Using JavaScript config files when CSS-native works
- ❌ Barrel exports that break tree-shaking
- ❌ Hardcoding colors in components instead of using tokens
- ❌ Publishing without provenance attestation
- ❌ Classic npm tokens (will break after Nov 2025)
- ❌ CSS preprocessors (Sass/Less incompatible with Tailwind 4.x)

---

## Next Steps

1. **Immediate**: Create root `package.json` and `pnpm-workspace.yaml`
2. **Short-term**: Add GitHub Actions workflow for CI/CD with OIDC npm publishing
3. **Medium-term**: Add Vitest test suites for all components
4. **Documentation**: Create README files with installation and usage examples
5. **Tooling**: Consider Storybook for visual documentation
6. **Versioning**: Implement Changesets for monorepo version management

---

## Relations
- implements [[self-learning-agent-protocol]]
- creates [[shadcn-tailwind-primer]]
- creates [[design-system-deployment-protocol]]
- informs [[recipe-lab]] (consuming application)