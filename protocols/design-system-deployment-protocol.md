---
title: Design System Deployment Protocol
created: 2026-01-01
modified: 2026-02-16
type: instruction
entity_type: instruction
status: draft
permalink: protocols/design-system-deployment-protocol
tags:
- project/agent-kb
- domain/design-system
- domain/deployment
- domain/ci-cd
- status/draft
- op/update-node
---

# Design System Deployment Protocol

**Status**: draft
**Last Updated**: 2026-02-16

## Purpose

Automate npm package publishing for @braisenly design system packages via GitHub Actions with OIDC trusted publishing.

---

## Prerequisites

- [ ] npm organization configured (@braisenly)
- [ ] GitHub repository with Actions enabled
- [ ] pnpm-workspace.yaml at root
- [ ] Root package.json with workspace scripts

---

## GitHub Actions Workflow

### `.github/workflows/publish.yml`

```yaml
name: Publish Packages

on:
  push:
    branches: [main]
    paths:
      - 'packages/**'

permissions:
  contents: read
  id-token: write  # Required for OIDC

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v4
        with:
          version: 9
          
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          registry-url: 'https://registry.npmjs.org'
          
      - run: pnpm install --frozen-lockfile
      - run: pnpm -r build
      - run: pnpm -r test
      
      - run: pnpm -r publish --access public --no-git-checks
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

## npm OIDC Setup

1. Go to npmjs.com → Access Tokens → Generate New Token
2. Select "Granular Access Token" with publish scope
3. Configure trusted publisher for GitHub Actions
4. Add token as `NPM_TOKEN` secret in GitHub repo

**WARNING**: Classic tokens deprecated, revoked November 19, 2025!

---

## Version Management

### Option A: Changesets (Recommended for Monorepo)
```bash
pnpm add -Dw @changesets/cli
pnpm changeset init
```

### Option B: Manual Versioning
Update package.json versions manually before merge to main.

---

## Pre-publish Checklist

- [ ] All tests passing
- [ ] TypeScript compiles without errors
- [ ] Build outputs generated in dist/
- [ ] Version bumped appropriately
- [ ] CHANGELOG updated

---

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-02-16 | Metadata normalization and relation format cleanup | Aligned note with KB frontmatter contract and standardized relation syntax for graph consistency. | KB hygiene sweep for protocol notes. |
| 2026-02-16 | Relation keyword normalization | Converted relation lines to canonical `verb [Target]` style to keep relation typing consistent. | Follow-up graph hygiene pass. |

## Relations

- implements [[shadcn-tailwind-design-system-founding-document]]
- follows [[shadcn-tailwind-primer]]