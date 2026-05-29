---
created: 2026-04-19
modified: 2026-05-28
entity_type: note
status: archived
title: 2026-04-19 - Claude user-scope MCP install and verification
type: note
permalink: kb/sessions/2026-04-19-claude-user-scope-mcp-install-and-verification
tags:
- claude-code
- mcp
- user-scope
- verification
- operations
---

# 2026-04-19 - Claude user-scope MCP install and verification

## Purpose
Document the end-to-end installation, correction, and verification of Claude Code MCP servers sourced from `~/.agents/settings/global/claude-mcpServers.json`, with explicit proof that Claude sees them as **user-scoped** servers.

## Initial User Ask
Install all MCP server settings for Claude Code from `.agents` into the Claude user configuration path, specifically asking for the Windows `~/.claude/settings.json` target "wherever the command wants to put them."

## Initial Mistake
The first pass merged the `mcpServers` block into `~/.claude/settings.json`.

That was incorrect for Claude Code user-scoped MCP configuration.

## Documentation Basis
Used official Claude Code docs via Context7.

Verified points:
- `claude mcp add --scope user ...` writes user-scoped MCP configuration to `~/.claude.json`.
- `~/.claude/settings.json` is not the documented user MCP store.
- `claude mcp get <name>` and `claude mcp list` are the supported CLI inspection commands for configured MCP servers.
- `/mcp` inside Claude Code is the interactive command for viewing active MCP servers.

## Corrective Action
1. Verified local Claude CLI availability.
2. Backed up `~/.claude.json`.
3. Backed up `~/.claude/settings.json`.
4. Used `claude mcp add-json --scope user <name> <json>` to install stdio-configured servers from `~/.agents/settings/global/claude-mcpServers.json`.
5. Used `claude mcp add --transport http exa --scope user 'https://mcp.exa.ai/mcp?exaApiKey='` for `exa` because `add-json` rejected the raw URL-only payload.
6. Removed the mistaken `mcpServers` key from `~/.claude/settings.json`.

## Files Touched
- `/home/luke/.claude.json`
- `/home/luke/.claude/settings.json`

## Backup Artifacts
- `/home/luke/.claude.json.bak-20260419-005903`
- `/home/luke/.claude/settings.json.bak-20260419-005903-prescopefix`
- `/home/luke/.claude/settings.json.bak-20260418-194543`

## Installed User-Scoped MCP Servers
Claude user config now contains these 16 MCP server entries:
- `augment-context-engine`
- `basic-memory`
- `chrome-devtools`
- `context7`
- `docker-mcp`
- `exa`
- `fetch`
- `filesystem`
- `github`
- `iterm-mcp`
- `mcp-obsidian`
- `next-devtools`
- `postgres`
- `puppeteer`
- `sequentialthinking`
- `serena`

## Direct CLI Evidence
The final verification step used Claude's own CLI against **user settings only**.

Verification command pattern:
```bash
claude --setting-sources user mcp get <name>
```

Representative positive evidence:

```text
github:
  Scope: User config (available in all your projects)
  Status: ✓ Connected
```

```text
exa:
  Scope: User config (available in all your projects)
  Status: ✓ Connected
  Type: http
  URL: https://mcp.exa.ai/mcp?exaApiKey=
```

```text
context7:
  Scope: User config (available in all your projects)
  Status: ✓ Connected
```

These CLI responses constitute direct proof that Claude sees the installed servers from **user scope**.

## Full Claude-Reported State
Results from `claude --setting-sources user mcp get <name>` across the full installed set:

- `augment-context-engine` — Scope: User config; Status: Failed to connect
- `basic-memory` — Scope: User config; Status: Failed to connect
- `chrome-devtools` — Scope: User config; Status: Connected
- `context7` — Scope: User config; Status: Connected
- `docker-mcp` — Scope: User config; Status: Connected
- `exa` — Scope: User config; Status: Connected
- `fetch` — Scope: User config; Status: Connected
- `filesystem` — Scope: User config; Status: Failed to connect
- `github` — Scope: User config; Status: Connected
- `iterm-mcp` — Scope: User config; Status: Connected
- `mcp-obsidian` — Scope: User config; Status: Failed to connect
- `next-devtools` — Scope: User config; Status: Connected
- `postgres` — Scope: User config; Status: Connected
- `puppeteer` — Scope: User config; Status: Failed to connect
- `sequentialthinking` — Scope: User config; Status: Connected
- `serena` — Scope: User config; Status: Connected

## Proven Outcomes
- The Claude user-scope MCP install is now in the correct file: `~/.claude.json`.
- Claude itself resolves all 16 configured servers from **user scope**.
- 11 servers currently connect successfully in this environment.
- 5 servers are registered correctly but currently fail connection health checks.
- `~/.claude/settings.json` no longer contains the incorrect `mcpServers` payload.

## Remaining Follow-Up
If needed, the next operational step is not installation but remediation of the 5 failing servers:
- `augment-context-engine`
- `basic-memory`
- `filesystem`
- `mcp-obsidian`
- `puppeteer`

## Relations
- related_to [[Manual MCP Client Operation for Codex]]
- related_to [[Index – MCP and Codex Operations]]
- related_to [[Basic Memory Semantic Reindex Protocol]]
