---
title: Per-Assistant MCP Schemas — dot-agents
type: reference
permalink: agent-kb/projects/dot-agents/per-assistant-mcp-schemas-dot-agents
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/dot-agents
- domain/mcp
- domain/config-management
- op/reference
---

# Per-Assistant MCP Schemas — dot-agents

Field-by-field reference for every coding assistant's MCP config format, plus translation rules from the canonical master config.

Full runbook with code examples: `~/.agents/docs/MASTER_MCP_INSTRUCTIONS.md`

---

## Quick Translation Matrix

| Concept | Master (OpenCode) | Claude Code | Cursor | Gemini CLI | Codex |
|---|---|---|---|---|---|
| Root key | `mcp` | `mcpServers` | `mcpServers` | `mcpServers` | `[mcp_servers."name"]` |
| Format | JSONC | JSON | JSON | JSON | TOML |
| Command | `["cmd","arg"]` (array) | `"cmd"` + `["arg"]` | `"cmd"` + `["arg"]` | `"cmd"` + `["arg"]` | `"cmd"` + `["arg"]` |
| Env key | `environment` | `env` | `env` | `env` | `[.env]` subtable |
| Env syntax | `{env:VAR}` | `${VAR}` | `${env:VAR}` | `$VAR` | `""` (shell) |
| Disable | `enabled: false` | `disabledMcpServers[]` | omit server | omit server | `enabled = false` |
| Timeout | ms | not supported | not supported | ms | seconds |
| HTTP endpoint | `url` | `url` | `url` | **`httpUrl`** | `url` |
| Bearer auth | `headers.Authorization` | same | same | same | `bearer_token_env_var` |

---

## Claude Code

**Config:** `~/.claude/settings.json` → symlink → `~/.agents/settings/global/claude-code-global.json`  
**Root:** `mcpServers`  
**Disable mechanism:** top-level `disabledMcpServers: ["name"]` array (no per-server `enabled`)

### Translation rules

| Master | Claude Code |
|---|---|
| `command[0]` | `"command": "<value>"` (string) |
| `command[1:]` | `"args": [...]` |
| `"environment"` key | `"env"` key |
| `{env:VAR}` | `${VAR}` |
| `"enabled": false` | add `"<name>"` to `disabledMcpServers[]` |
| `"type": "local"` | omit or `"stdio"` |
| `"type": "remote"` | `"http"` or `"sse"` |
| `"timeout"` | drop (not supported) |
| `"oauth"` | drop (not supported) |

---

## OpenCode

**Config:** `~/.config/opencode/opencode.jsonc` → symlink → `~/.agents/settings/global/opencode.jsonc`  
**Root:** `mcp`  
**Disable mechanism:** `"enabled": false` per server

### Translation rules

Master config IS OpenCode format. Copy verbatim.

---

## Cursor

**Config:** Per-project `.cursor/mcp.json` (distributed via `dot-agents add`, hard-linked)  
**Root:** `mcpServers`  
**Disable mechanism:** omit the server entirely (no `enabled` field, no disabled list)

### Hard-link warning

Cursor configs added via `dot-agents add` are **hard links** to `~/.agents/platform/cursor/`.
Editing the file in a project directly modifies the central source.

### Translation rules

| Master | Cursor |
|---|---|
| `command[0]` | `"command": "<value>"` |
| `command[1:]` | `"args": [...]` |
| `"environment"` key | `"env"` key |
| `{env:VAR}` | `${env:VAR}` (add `env:` prefix) |
| `"enabled": false` | omit server entirely |
| `"type": "local"` | omit `type` |
| `"type": "remote"` | `"type": "sse"` or `"http"` |
| `"timeout"` | drop |
| `"oauth"` | `"auth": { "type": "oauth", ... }` (different shape) |

---

## Gemini CLI

**Config:** `~/.gemini/settings.json` → sourced from `~/.agents/settings/global/gemini-settings.json`  
**Root:** `mcpServers`  
**Disable mechanism:** omit the server entirely (no `enabled` field)

### HTTP vs SSE distinction (Gemini-specific)

- SSE remote endpoint → `"url": "..."`
- Plain HTTP remote endpoint → **`"httpUrl": "..."`** (NOT `"url"`)

### Translation rules

| Master | Gemini CLI |
|---|---|
| `command[0]` | `"command": "<value>"` |
| `command[1:]` | `"args": [...]` |
| `"environment"` key | `"env"` key |
| `{env:VAR}` | `$VAR` (strip `{env:` and `}`) |
| `"enabled": false` | omit server entirely |
| `"type": "local"` | omit `type` |
| remote + HTTP | `"httpUrl": "..."` |
| remote + SSE | `"url": "..."` |
| `"headers"` values | same, but `$VAR` syntax |
| `"timeout"` | `"timeout"` (same ms unit, default 600000) |
| `"oauth"` | `"authProviderType"` + `"oauth"` object |

---

## Codex

**Config:** `~/.codex/config.toml` → sourced from `~/.agents/settings/global/codex-config.toml`  
**Root:** `[mcp_servers."<name>"]` TOML section per server  
**Disable mechanism:** `enabled = false` per server

### TOML format example

```toml
[mcp_servers."basic-memory"]
command = "uvx"
args = ["basic-memory", "mcp"]
startup_timeout_sec = 900
enabled = true

[mcp_servers."basic-memory".env]
# No env vars needed for this server
```

### Translation rules

| Master | Codex |
|---|---|
| `command[0]` | `command = "<value>"` (string) |
| `command[1:]` | `args = [...]` (TOML array) |
| `"environment"` section | `[.env]` subtable |
| `{env:VAR}` value | `KEY = ""` (empty string; actual value from shell) |
| `"enabled"` | `enabled = true/false` |
| `"timeout" / 1000` | `startup_timeout_sec = <int>` (seconds, not ms) |
| `"type": "local"` | omit `url` |
| `"type": "remote"` | `url = "..."` |
| `headers.Authorization: "Bearer {env:X}"` | `bearer_token_env_var = "X"` (just the var name) |
| `"oauth"` | not supported |

---

## Relations

- [[MCP Management System — dot-agents]] — architecture overview and operations
- [[dot-agents CLI — Agentic Usage Guide]] — CLI danger zones and safe patterns
- [[dot-agents Git Workflow]] — commit discipline