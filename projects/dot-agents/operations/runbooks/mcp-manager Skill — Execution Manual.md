---
title: mcp-manager Skill — Execution Manual
type: reference
permalink: agent-kb/projects/dot-agents/operations/runbooks/mcp-manager-skill-execution-manual
created: 2026-04-19
modified: 2026-04-19
entity_type: reference
status: evergreen
tags:
- project/dot-agents
- domain/mcp
- domain/config-management
- op/manage
---

# mcp-manager Skill — Execution Manual

**Agent retrieval framing:** Load this note before running or reasoning about any `/mcp-manager` operation. It maps every file the skill reads and writes, every shell command it runs, and every field transformation it applies. Use it to pre-flight the exact blast radius of `add-mcp` or `add-assistant` before executing.

Skill file on disk: `~/.agents/claude/skills/mcp-manager/SKILL.md`
Filesystem manual: `~/.agents/docs/mcp-manager-manual.md`

---

## Bootstrap (every invocation — reads only)

| File | Full path | Why |
|---|---|---|
| Master config | `~/.agents/mcp/master-mcp.jsonc` | Ground-truth 26-server list in OpenCode schema |
| Instructions runbook | `~/.agents/docs/MASTER_MCP_INSTRUCTIONS.md` | Per-assistant translation algorithms + field maps |
| Server index | `~/.agents/settings/global/mcp-master.md` | Enable/disable status per assistant |

---

## add-mcp — Blast Radius

6 files written atomically, then one commit.

| # | File | Operation | Key transformation |
|---|---|---|---|
| 1 | `mcp/master-mcp.jsonc` | append entry | OpenCode schema + JSONC comments |
| 2 | `settings/global/mcp-master.md` | append table row | Mark ✅/❌ per assistant column |
| 3 | `settings/global/claude-code-global.json` | add `mcpServers.<name>` key | `command[]→string+args`, `environment→env`, `{env:V}→${V}`, `enabled:false→disabledMcpServers[]` |
| 4 | `settings/global/opencode.jsonc` | add `mcp.<name>` key | verbatim copy (OpenCode IS canonical) |
| 5 | `settings/global/gemini-settings.json` | add `mcpServers.<name>` key | `{env:V}→$V`, HTTP remote uses `httpUrl` not `url`, disabled = omit entirely |
| 6 | `settings/global/codex-config.toml` | append `[mcp_servers."name"]` sections | `timeout/1000→startup_timeout_sec`, env values = `""` from shell, bearer auth → `bearer_token_env_var` |

Commit message: `feat(mcp): add <name> server`

### Non-obvious edge cases

- **Claude Code** has no per-server `enabled` field. `enabled:false` → add server name to top-level `disabledMcpServers[]` array; do not add a `mcpServers` entry at all.
- **Gemini** plain-HTTP remote endpoints must use `"httpUrl"` key, NOT `"url"`. SSE remotes use `"url"`. This distinction is Gemini-specific.
- **Codex** timeout is in **seconds**: `timeout_ms / 1000 = startup_timeout_sec`. Default 900000ms → 900 sec.
- **Codex** env vars: set `KEY = ""` in the `[.env]` subtable; actual value is read from shell environment at runtime.
- **OpenCode** `command` is an array (full argv). All other assistants split it: `command[0]→command string`, `command[1:]→args array`.

---

## add-assistant — File Writes

| Step | File / path created | Notes |
|---|---|---|
| 5 | `platform/<name>/` (dir) | Even if empty; add hooks/ and rules/ subdirs if supported |
| 6 | `settings/global/<name>-config.<ext>` | Translate all enabled servers from master |
| 7 | `settings/global/mcp-master.md` | Add column to Canonical Server List table |
| 8 | `docs/MASTER_MCP_INSTRUCTIONS.md` | Add `## 1.N <Assistant>` section: field ref + translation table |
| 9 | `<assistant-config-path>` | Symlink if assistant reads from a fixed path |
| 12 | KB note in agent-kb | `projects/dot-agents/<name>-integration.md` via Curator |

Commit: `feat(assistant): onboard <name>`

---

## status — Reads & Shell Commands

**Files read (no writes):**

| File | JSON/TOML path extracted |
|---|---|
| `mcp/master-mcp.jsonc` | all server keys |
| `settings/global/claude-code-global.json` | `$.mcpServers` keys minus `$.disabledMcpServers` |
| `settings/global/opencode.jsonc` | `$.mcp` where `enabled === true` |
| `settings/global/gemini-settings.json` | `$.mcpServers` keys (all present = enabled) |
| `settings/global/codex-config.toml` | `[mcp_servers.*]` where `enabled = true` |

**Shell commands executed (from `~/.agents/`):**

```bash
# Credential check
grep -rn 'ghp_\|sk-\|tvly-\|ctx7sk-' \
  settings/global/claude-code-global.json \
  settings/global/opencode.jsonc \
  settings/global/gemini-settings.json \
  settings/global/codex-config.toml

# Last-commit per file
for f in settings/global/claude-code-global.json \
          settings/global/opencode.jsonc \
          settings/global/gemini-settings.json \
          settings/global/codex-config.toml; do
  echo "$f: $(git log --oneline -1 -- $f)"
done
```

**Drift matrix legend:** ✅ present+enabled · ❌ present but disabled · ⚠️ in master but missing · — intentionally N/A

---

## Translation Quick-Reference

| Field | OpenCode (master) | Claude Code | Gemini | Codex |
|---|---|---|---|---|
| Env syntax | `{env:VAR}` | `${VAR}` | `$VAR` | `""` (shell) |
| Disable | `enabled:false` | `disabledMcpServers[]` | omit entry | `enabled=false` |
| Timeout | `900000` ms | not supported | `60000` ms default | `startup_timeout_sec=900` |
| HTTP remote | `"url"` | `"url"` | **`"httpUrl"`** | `url=` |
| Command | `["cmd","a"]` array | `"cmd"` + `["a"]` | `"cmd"` + `["a"]` | `cmd` + `["a"]` |
| Env key | `"environment"` | `"env"` | `"env"` | `[.env]` subtable |

---

## Symlink Map

| Repo file | Live path | Who reads it |
|---|---|---|
| `settings/global/claude-code-global.json` | `~/.claude/settings.json` | Claude Code |
| `settings/global/opencode.jsonc` | `~/.config/opencode/opencode.jsonc` | OpenCode |

Gemini and Codex have no symlinks — their live configs must be manually synced from repo copies.

---

## Relations

- Implements [[MCP Management System — dot-agents]]
- Derived from [[Per-Assistant MCP Schemas — dot-agents]]
- Related to [[dot-agents CLI — Agentic Usage Guide]]