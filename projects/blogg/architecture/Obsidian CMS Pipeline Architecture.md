---
title: Obsidian CMS Pipeline Architecture
type: reference
permalink: projects/blogg/architecture/obsidian-cms-pipeline
entity_type: reference
status: evergreen
tags:
- project/blogg
- domain/architecture
- domain/obsidian
---

# Obsidian CMS Pipeline Architecture

Dual-mode CMS pipeline bridging a local Obsidian Vault with the Next.js application. Supports both static build-time content (Contentlayer) and runtime dynamic content (live reads from vault).

## Pipeline Flow

```
Obsidian Vault --> scripts/blog-watcher.js (Chokidar)
                      |
                      +--> 1. scripts/ingest-obsidian.mjs --> data/blog/ (MDX) --> Contentlayer --> Static/ISR
                      |
                      +--> 2. POST /api/revalidate-blog --> Runtime Cache Purge --> lib/runtime-blog.ts --> Dynamic
```

## Components

### A. Watcher (`scripts/blog-watcher.js`)
- Watches `~/vaults/obsidian/blog` for add/edit/delete
- Triggers ingestion + cache revalidation

### B. Ingestion Layer (`scripts/ingest-obsidian.mjs`)
- Transforms WikiLinks (`[[Link]]`) to standard Markdown
- Copies and rewrites image paths (`![[img.png]]`)
- Writes to `data/blog/` for Contentlayer processing
- Integrates with theme features: Search, RSS, Tags, TOC

### C. Runtime Layer (`lib/runtime-blog.ts`)
- Direct vault filesystem reads at request time
- Uses Next.js `experimental.useCache` + `cacheTag('posts')`
- Zero-latency updates (no Contentlayer regeneration wait)

## Configuration

`scripts/obsidian-config.json`:
```json
{
  "vaultPath": "/Users/luke/vaults/obsidian",
  "postsFolder": "blog",
  "publishTag": "publish"
}
```

## Usage

- **Dev**: `pnpm dev` + `pnpm blog:watch` (two terminals)
- **Prod**: Run `pnpm ingest` during build. Vault must be accessible (mounted volume) for runtime layer.

## Relations

- child_of [[Next.js Blog (blogg)]]
- implements [[Contentlayer]]
- uses [[Obsidian]]
- related_to [[Obsidian MCP Operations]]
