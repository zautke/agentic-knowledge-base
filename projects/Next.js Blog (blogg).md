---
title: Next.js Blog (blogg)
type: reference
permalink: projects/blogg
entity_type: reference
status: evergreen
tags:
- project/blogg
- domain/web-application
- status/evergreen
aliases:
- blogg
- next15-blog-template
- nextjs-materialui-postgres-blog
---

# Next.js Blog (blogg)

A Next.js tech blog application with Material-UI v7, Tailwind CSS v4, Contentlayer2, and Docker deployment. Originally forked from timlrx/tailwind-nextjs-starter-blog, heavily customized with a Clarence design system, Obsidian CMS pipeline, MDX playground components, and MCP server integration.

## Project Overview

- **Type**: Next.js Web Application (software-lib archetype)
- **Repository**: github:zautke/next15-blog-template
- **Status**: Production (v1.0.9+)
- **Machine**: largo (MacBook, secondary dev), adagio (Windows, primary dev)
- **Local Path**: `/Volumes/FLOUNDER/dev/blogg`

## Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Framework | Next.js | 16.2.0-canary.13 |
| React | React | 19.3.0-canary |
| UI Library | Material-UI (MUI) | v7 |
| Utility CSS | Tailwind CSS | v4 |
| Content | Contentlayer2 | fork of contentlayer |
| Styling Engine | Emotion + Pigment CSS | latest |
| GraphQL (planned) | PostGraphile | - |
| Database (planned) | PostgreSQL | - |
| E2E Tests | Playwright | 1.55 |
| Unit Tests | Vitest + Storybook | 3.2 / 9.1 |
| Process Mgr | PM2 | latest |
| Container | Docker (Alpine) | multi-stage |
| CI/CD | GitHub Actions | semantic-release |
| Package Mgr | pnpm | 10.24.0 |

## Key Ports

| Service | Port |
|---------|------|
| Dev server | 4850 |
| Production Docker | 4800 |
| Storybook | 6006 |

## Development Commands

```bash
pnpm install          # Install dependencies
pnpm dev              # Dev server on port 4850
pnpm build            # Production build (STANDALONE=true)
pnpm start            # Start production server
pnpm storybook        # Component library on port 6006
pnpm test             # Playwright E2E tests
pnpm lint             # ESLint
```

## Architecture Highlights

- **App Router**: All routes in `app/` directory with React Server Components by default
- **Clarence Design System**: Custom component library in `clarence-components/`
- **Obsidian CMS Pipeline**: File watcher + ingestion from Obsidian vault to `data/blog/`
- **MDX Playground**: React Live-powered interactive code blocks
- **Multi-stage Docker**: base → development | builder → production
- **Dual Theme**: MUI 7 + Tailwind CSS v4 coexistence via `customTheme/`

## Documentation Map

All curated docs live in `docs/`:
- `docs/architecture/` — System design, flow diagrams
- `docs/guides/` — How-to guides, deployment, operations
- `docs/analysis/` — Research and performance analysis
- `docs/session-logs/` — Historical AI session records
- `docs/scratch/` — WIP snippets

## Known Issues / Technical Debt

1. `CLAUDE.md` (AGENTS.md) describes RecipeLab, not this project — needs rewrite
2. `README.md` is upstream template content — needs overhaul
3. Dual release tooling (semantic-release + release-please) on same branch
4. Stale artifact files (`.old`, `.orig`, `.disabled`) need cleanup
5. `pnpm test` maps to Playwright, not Vitest (documentation mismatch)

## Relations

- uses [[Material UI Design System]]
- uses [[Next.js]]
- uses [[Tailwind CSS]]
- uses [[Contentlayer]]
- uses [[Docker]]
- uses [[Playwright]]
- uses [[Storybook]]
- implements [[design-system-deployment-protocol]]
- child_of [[Fundamental – Agent KB]]