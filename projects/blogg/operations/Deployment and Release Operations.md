---
title: Deployment and Release Operations
type: reference
permalink: projects/blogg/operations/deployment
entity_type: reference
status: evergreen
tags:
- project/blogg
- domain/operations
- domain/docker
---

# Deployment and Release Operations

## Versioning

Semantic-release with Conventional Commits. Config: `.releaserc.json`.

**Known Issue:** Both semantic-release AND release-please are configured on `main` push. These are competing strategies. Recommend keeping semantic-release and removing release-please.

## CI Pipeline

GitHub Actions `release.yml`:
1. Semantic-release runs on push to main/next/beta/alpha
2. On success, builds and pushes Docker images to GHCR

## Docker Registry

- **Registry**: `ghcr.io/zautke/next15-blog-template/blog`
- **Tags**: `<version>`, `<version>-<shortsha>`, `latest`

## CLI Commands

```bash
pnpm docker:login       # Requires GH_TOKEN in env
pnpm docker:build       # Build image
pnpm docker:build:nc    # Build (no cache)
pnpm docker:push        # Push to GHCR
pnpm docker:tags        # List tags
pnpm tasks              # List available tasks
```

## Makefile Variables

Overrides via `.env` (key=val lines auto-included):
- `REGISTRY=ghcr.io`
- `OWNER` / `REPO` parsed from `git remote origin`
- `IMAGE_NAME=blog`
- `VERSION` from `package.json`
- `GIT_SHA` from `git rev-parse --short=8 HEAD`

## Docker Architecture

Multi-stage Dockerfile: `base` --> `development` | `builder` --> `production`

| Service | Port | Mode |
|---------|------|------|
| `blog` (compose.yml) | 4800:3000 | Production |
| `blog-dev` (compose.yml) | 4850:3000 | Development (volume mounts, file watching) |

PM2 manages the process in both modes (`pm2.js` / `pm2.dev.js`).

## Release History

Latest successful: v1.0.8 (2025-09-19). Docker image pulled, container running, health check passing.

## Relations

- child_of [[Next.js Blog (blogg)]]
- implements [[design-system-deployment-protocol]]
- uses [[Docker]]
- uses [[GitHub Actions]]