---
title: Documentation Audit and Tech Debt
type: reference
permalink: projects/blogg/planning/documentation-audit
entity_type: reference
status: draft
tags:
- project/blogg
- domain/planning
- meta/audit
---

# Documentation Audit and Tech Debt

**Audit Date:** 2026-02-14
**Status:** Audit complete. Remediation pending.

## Existing Docs (Curated)

All docs moved from project root to `docs/` sub-folders:
- `docs/architecture/` — 3 files (CMS pipeline, TUI design, MDX playground flow)
- `docs/guides/` — 6 files (deployment, storybook, TUI impl, gum patterns, codeblock, obsidian MCP)
- `docs/analysis/` — 2 files (CLI performance, Clarence components)
- `docs/session-logs/` — 3 files (historical AI session records)
- `docs/scratch/` — 2 files (middleware snippet, empty GEMINI.md)

## Missing Documentation

### CRITICAL (blocks onboarding)
- [ ] `docs/guides/GETTING_STARTED.md` — Prerequisites, setup, first run
- [ ] `docs/guides/ENVIRONMENT_VARIABLES.md` — Complete env var reference (20+ vars scattered across config)
- [ ] `README.md` overhaul — Currently upstream template content, not this project

### HIGH (important for contributors)
- [ ] `docs/architecture/API_ROUTES.md` — /api/health, /api/newsletter, /api/revalidate-blog
- [ ] `docs/guides/TESTING.md` — Playwright E2E + Vitest/Storybook strategy
- [ ] `docs/guides/CI_CD.md` — 5 GitHub Actions workflows, dual release tooling conflict
- [ ] `docs/guides/DOCKER_DEVELOPMENT.md` — Multi-stage Docker, compose services, PM2
- [ ] `docs/architecture/COMPONENTS.md` — Component hierarchy, layouts, theme coexistence
- [ ] `CONTRIBUTING.md` — Branch conventions, commit format, code style

### MEDIUM (nice to have)
- [ ] `docs/guides/SCRIPTS.md` — blog-watcher, ingest, render-mcp-config, postbuild, rss
- [ ] `docs/guides/MCP_CONFIGURATION.md` — mcp.jsonc.example, render script, Claude Desktop integration
- [ ] `docs/guides/CONTENT_AUTHORING.md` — Frontmatter schema, dual content paths, MDX playground usage
- [ ] `docs/architecture/ADR_INDEX.md` — Architecture Decision Records
- [ ] `docs/guides/TROUBLESHOOTING.md` — Common build errors, port conflicts, known issues
- [ ] `docs/architecture/TECH_STACK.md` — Categorized dependency reference

## Inconsistencies to Resolve

1. **CLAUDE.md (AGENTS.md) describes RecipeLab, not this blog** — Needs full rewrite
2. **`pnpm test` maps to Playwright, not Vitest** — Docs say Vitest
3. **Dual release tooling** — semantic-release + release-please both on main push
4. **Port discrepancy** — AGENTS.md says 4000, actual is 4850 (dev) / 4800 (prod)
5. **Stale artifact files** — .old, .orig, .disabled, working.Dockerfile, test-fix.js

## Relations

- child_of [[Next.js Blog (blogg)]]
- tracks [[Deployment and Release Operations]]
- tracks [[Storybook Component Manual]]