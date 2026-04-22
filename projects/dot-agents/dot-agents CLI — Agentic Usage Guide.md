---
title: dot-agents CLI — Agentic Usage Guide
type: reference
permalink: agent-kb/projects/dot-agents/dot-agents-cli-agentic-usage-guide
created: 2026-04-18
modified: 2026-04-18
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/tooling
- domain/dot-agents
- op/reference
---

# dot-agents CLI — Agentic Usage Guide

## Purpose

`dot-agents` is a **centralized AI coding-assistant config manager**. It maintains one source of truth in `~/.agents/` and distributes configs to individual projects via symlinks (Claude Code, OpenCode, Codex) or hard links (Cursor).

**Repo:** `/Users/luke/.agents/dot-agents` (v0.1.9, Bash)  
**Config home:** `/Users/luke/.agents/`  
**Entry point:** `dot-agents <command>`

## What It Reads / Writes

### Reads
- `~/.agents/config.json` — project registry
- `~/.agents/rules/`, `~/.agents/settings/`, `~/.agents/mcp/`, `~/.agents/skills/`
- `.claude/`, `.cursor/` in target projects

### Writes (per command)
| Target | What | How |
|--------|------|-----|
| `~/.agents/config.json` | Project registration | `jq` modification |
| `~/.claude/settings.json` | **Global** Claude settings | Symlink → `~/.agents/settings/global/claude-code-global.json` |
| `<project>/.claude/rules/` | Claude rules | Symlinks (`ln -sf`) |
| `<project>/.claude/settings.local.json` | Project settings | Symlink |
| `<project>/.mcp.json` | MCP config | Symlink |
| `<project>/.cursor/rules/` | Cursor rules | **HARD LINKS** (`ln -f`) |
| `<project>/.cursor/settings.json` | Cursor settings | Hard link |
| `<project>/.cursor/mcp.json` | Cursor MCP | Hard link |
| `<project>/.opencode/agent/` | OpenCode agents | Symlinks |

**Critical:** `opencode_create_links()` creates project-level `opencode.json` symlinks — it does NOT touch `~/.config/opencode/opencode.jsonc`.

## Git Operations (isolated to `~/.agents/`)

| Command | Git op |
|---------|--------|
| `sync init` | `git init` in `~/.agents/` |
| `sync commit` | `git add .` + `git commit` |
| `sync push` | `git push -u origin <branch>` |
| `sync pull` | `git pull` (plain, no --rebase) |

**The CLI does NOT run git in project directories. It cannot corrupt project git history.**

## Safe Command Sequence

```bash
# Before any dot-agents operation:
dot-agents doctor          # health check — read-only
git -C ~/.agents status    # verify clean working tree

# Add a project:
dot-agents add /path/to/project

# After any dot-agents command:
git -C ~/.agents diff      # verify only expected changes
dot-agents sync commit     # commit with explicit message (never git add .)
```

## Danger Zones

| Command | Risk | Mitigation |
|---------|------|-----------|
| `add --force` | Overwrites project configs (backs up to `.dot-agents-backup`) | Only use if you know what's being overwritten |
| `remove --clean` | `rm -rf` platform dirs in `~/.agents/` | Always `doctor` first |
| `init --force` | Replaces `~/.claude/settings.json` symlink globally | Affects ALL Claude Code sessions on machine |
| Cursor hard links | Editing `.cursor/rules/*.mdc` in project modifies `~/.agents/rules/` | Never edit Cursor rules in project dirs directly |
| `sync pull` | Plain `git pull` creates merge commits | Prefer `git fetch + rebase` manually |

## Global Symlink Created by `dot-agents init`

```
~/.claude/settings.json → ~/.agents/settings/global/claude-code-global.json
```

This symlink affects every Claude Code session on the machine. If `claude-code-global.json` becomes malformed, all sessions fail silently or lose hooks.

## Template vs Active Config

| File | Purpose |
|------|---------|
| `~/.agents/dot-agents/src/share/templates/standard/` | Default templates for new installs |
| `~/.agents/settings/global/` | Canonical source for all assistant configs |
| `~/.config/opencode/opencode.jsonc` | OpenCode active config — should be copied/symlinked from repo source |

## Relations
- Explains [[MCP Master List — dot-agents]]
- Used by [[dot-agents Git Workflow]]
- Depends on [[Agentic Knowledge Base System Overview]]