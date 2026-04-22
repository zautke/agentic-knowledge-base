---
title: Development Operations
type: reference
permalink: kb/projects/indexer/operations/development-operations
tags:
- project/indexer
- domain/operations
- status/draft
---

# Development Operations

Operational note for day-to-day development and verification on the `indexer` repository.

## Working Locations
- Main repository: `/Volumes/FLOUNDER/dev/indexer`
- Logger worktree: `/Volumes/FLOUNDER/dev/indexer-logger`
- Infrastructure worktree: `/Volumes/FLOUNDER/dev/indexer/.worktrees/infra`
- KB backfill worktree: `/Volumes/FLOUNDER/dev/indexer/.worktrees/indexer-kb-backfill`

## Common Commands
```bash
pnpm install
pnpm test
pnpm build
pnpm typecheck
```

UI-specific:
```bash
cd ui
pnpm install
pnpm dev
pnpm build
```

## Known Verification Constraints
- In the KB-backfill worktree, `pnpm test` is currently blocked before repo tests run because Vite/PostCSS loads `/Volumes/FLOUNDER/dev/postcss.config.mjs` and cannot resolve `@tailwindcss/postcss`.
- Current implementation claims around ingestion rewrite and evaluation persistence often require code inspection because the repo has known environment/config noise.

## Current Operational Focus
- ingestion rewrite validation
- evaluation storage/schema changes
- architecture visualization UI changes
- worktree-specific logger and infrastructure branches

## Related Notes
- [[System Architecture Overview]]
- [[Repository Documentation Map]]
- [[Implementation State - 2026-03-13]]
