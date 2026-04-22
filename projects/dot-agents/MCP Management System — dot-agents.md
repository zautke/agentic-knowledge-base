---
title: MCP Management System — dot-agents
type: reference
permalink: agent-kb/projects/dot-agents/mcp-management-system-dot-agents
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/dot-agents
- domain/mcp
- domain/config-management
- op/manage
---

# MCP Management System — dot-agents

Authoritative guide for managing MCP server configurations across all coding assistants in the `~/.agents` system.

## Architecture

```
~/.agents/
  mcp/
    master-mcp.jsonc          ← CANONICAL SOURCE (OpenCode schema with comments)
  settings/global/
    mcp-master.md             ← Per-assistant enable/disable index (table)
    claude-code-global.json   ← Claude Code MCP config (symlinked from ~/.claude/settings.json)
    opencode.jsonc            ← OpenCode MCP config (symlinked from ~/.config/opencode/opencode.jsonc)
    gemini-settings.json      ← Gemini CLI MCP config
    codex-config.toml         ← Codex MCP config
  docs/
    MASTER_MCP_INSTRUCTIONS.md ← Full runbook: schemas, translations, operations
  claude/skills/
    mcp-manager/SKILL.md      ← Claude Code skill: add-mcp, add-assistant, status
```

## Single Source of Truth Flow

```
mcp/master-mcp.jsonc
       │
       ├──translate──→ settings/global/claude-code-global.json
       ├──copy──────→ settings/global/opencode.jsonc  (OpenCode IS the canonical format)
       ├──translate──→ settings/global/gemini-settings.json
       └──translate──→ settings/global/codex-config.toml
```

Every change starts at `mcp/master-mcp.jsonc` and propagates to all assistant configs in the same commit.

## Master Config Schema (OpenCode / canonical)

```jsonc
{
  "<server-name>": {
    "type": "local" | "remote",
    "command": ["cmd", "arg1", ...],     // local only; array
    "environment": { "KEY": "{env:VAR}" }, // local; env var references
    "url": "https://...",                // remote only
    "headers": { "Authorization": "Bearer {env:VAR}" }, // remote auth
    "oauth": false,                      // remote; OAuth flow
    "enabled": true,                     // per-server toggle
    "timeout": 900000                    // ms
  }
}
```

## Enabled Servers (26 total, 2026-04-18)

| Server | Type | Default |
|---|---|---|
| augment-context-engine | local | ✅ |
| auggie-code | remote | ✅ |
| auggie-context | local | ✅ |
| basic-memory | local | ✅ |
| chrome-devtools | local | ✅ |
| context7 | remote | ✅ |
| codemunch | local | ✅ |
| docmunch | local | ✅ |
| docker-mcp | local | ✅ |
| exa | remote | ✅ |
| filesystem | local | ✅ |
| github | local | ✅ |
| iterm-mcp | local | ✅ |
| mcp-obsidian | local | ✅ |
| mcp_excalidraw | local | ✅ |
| next-devtools | local | ✅ |
| postgres | local | ✅ |
| tavily | remote | ✅ |
| fetch | local | ❌ disabled |
| sequentialthinking | local | ❌ disabled |
| BrowserTools | local | ❌ disabled |
| playwright | local | ❌ disabled |
| storybook-mcp | remote | ❌ disabled |
| supabase | remote | ❌ disabled |
| XcodeBuildMCP | local | ❌ disabled |

## Operations

### Add a server (`/mcp-manager add-mcp`)

1. Append entry to `mcp/master-mcp.jsonc` (OpenCode schema + comments)
2. Add row to `settings/global/mcp-master.md`
3. Translate + write to all 4 assistant configs (see [[Per-Assistant MCP Schemas — dot-agents]])
4. Commit: `feat(mcp): add <name> server`

### Onboard an assistant (`/mcp-manager add-assistant`)

1. Identify config path, root key, field names, env syntax, enable mechanism
2. Create `platform/<name>/` + `settings/global/<name>-config.<ext>`
3. Translate all enabled servers from master
4. Update `mcp-master.md` + `MASTER_MCP_INSTRUCTIONS.md`
5. Commit: `feat(assistant): onboard <name>`

### Audit drift (`/mcp-manager status`)

Read all 4 assistant configs + master → build ✅/❌/⚠️ matrix → check for hardcoded credentials → show last-commit per config file.

- Full execution manual: [[mcp-manager Skill — Execution Manual]]

## Required Environment Variables

| Variable | Server |
|---|---|
| `AUGMENT_API_KEY` | auggie-code |
| `AUGMENT_SESSION_AUTH` | auggie-context |
| `EXA_API_KEY` | exa |
| `GITHUB_PERSONAL_ACCESS_TOKEN` | github |
| `OBSIDIAN_API_KEY` | mcp-obsidian |
| `OBSIDIAN_BASE_URL` | mcp-obsidian |
| `OBSIDIAN_VAULT` | mcp-obsidian |
| `TAVILY_API_KEY` | tavily |

## Drift Prevention Rules

1. Every MCP change goes to `mcp/master-mcp.jsonc` first
2. All 4 assistant configs updated in the same commit
3. Never edit assistant configs outside `~/.agents/`
4. Never hardcode credentials — always `{env:VAR}` / `$VAR` / empty-string-from-shell

## Relations

- [[dot-agents CLI — Agentic Usage Guide]] — CLI used to distribute configs via symlinks/hard-links
- [[Per-Assistant MCP Schemas — dot-agents]] — field maps and translation tables for each assistant
- [[dot-agents Git Workflow]] — commit discipline for this repo
- [[Index – MCP and Codex Operations]] — upstream MCP protocol knowledge