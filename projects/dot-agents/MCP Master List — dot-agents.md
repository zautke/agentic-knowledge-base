---
title: MCP Master List — dot-agents
type: reference
permalink: agent-kb/projects/dot-agents/mcp-master-list-dot-agents
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/mcp
- domain/dot-agents
- op/reference
---

# MCP Master List — dot-agents

Canonical reference for all MCP servers. File on disk: `~/.agents/settings/global/mcp-master.md`.

**Rule: Any new MCP server goes here AND into all assistant configs in the same commit.**

## Required Environment Variables

| Variable | Server |
|----------|--------|
| `AUGMENT_API_KEY` | auggie-code |
| `AUGMENT_SESSION_AUTH` | auggie-context |
| `EXA_API_KEY` | exa |
| `GITHUB_PERSONAL_ACCESS_TOKEN` | github |
| `OBSIDIAN_API_KEY` | mcp-obsidian |
| `OBSIDIAN_BASE_URL` | mcp-obsidian |
| `OBSIDIAN_VAULT` | mcp-obsidian |
| `OLLAMA_API_KEY` | ollama provider (opencode) |
| `TAVILY_API_KEY` | tavily |

## Canonical Servers (as of 2026-04-18)

| Name | Type | OpenCode | Gemini | Claude Code |
|------|------|----------|--------|-------------|
| `augment-context-engine` | local: `auggie --mcp --mcp-auto-workspace` | ✅ | ✅ | ✅ |
| `auggie-code` | remote: `api.augmentcode.com/mcp` | ✅ | — | ✅ |
| `auggie-context` | local: `npx -y auggie-context-mcp@latest` | ✅ | — | ✅ |
| `basic-memory` | local: `uvx basic-memory mcp` | ✅ | ✅ | ✅ |
| `chrome-devtools` | local: `npx -y chrome-devtools-mcp@latest` | ✅ | ✅ | ✅ |
| `context7` | remote: `mcp.context7.com/mcp` | ✅ | ✅ | ✅ |
| `codemunch` (jcodemunch) | local: `uvx jcodemunch-mcp` | ✅ | ✅ | ✅ |
| `docmunch` (jdocmunch) | local: `uvx jdocmunch-mcp` | ✅ | ✅ | — |
| `docker-mcp` | local: `~/.local/bin/uvx docker-mcp` | ✅ | ✅ | ✅ |
| `exa` | remote: `mcp.exa.ai` | ✅ | ✅ | ✅ |
| `fetch` | local: `uvx mcp-server-fetch` | 🚫 | — | — |
| `filesystem` | local: `npx -y @modelcontextprotocol/server-filesystem` | ✅ | ✅ | ✅ |
| `github` | local: `npx -y @modelcontextprotocol/server-github` | ✅ | ✅ | ✅ |
| `iterm-mcp` | local: `npx -y iterm-mcp` | ✅ | ✅ | ✅ |
| `mcp-obsidian` | local: `uvx mcp-obsidian` | ✅ | ✅ | ✅ |
| `mcp_excalidraw` | local: `npx -y excalidraw-mcp` | ✅ | — | ✅ |
| `next-devtools` | local: `pnpm dlx next-devtools-mcp@latest` | ✅ | ✅ | ✅ |
| `postgres` | local: `npx -y @modelcontextprotocol/server-postgres` | ✅ | ✅ | ✅ |
| `sequentialthinking` | local: docker `mcp/sequentialthinking` | 🚫 | ✅ | 🚫 |
| `tavily` | remote: `mcp.tavily.com` | ✅ | — | ✅ |

## Env Var Reference Format by Assistant

| Assistant | Syntax |
|-----------|--------|
| OpenCode | `{env:VAR_NAME}` in `environment:` field |
| Gemini | Omit `env` field; inherit from shell (empty string = use system env) |
| Claude Code | Set in shell env; `claude-mcpServers.json` passes empty string |
| Codex | `""` in TOML env field; set via shell env |

## Relations
- Implements [[dot-agents CLI — Agentic Usage Guide]]
- Used by [[dot-agents Git Workflow]]