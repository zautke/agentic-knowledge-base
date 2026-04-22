---
title: Obsidian MCP Operations
type: reference
permalink: projects/blogg/operations/obsidian-mcp
entity_type: reference
status: evergreen
tags:
- project/blogg
- domain/operations
- domain/mcp
---

# Obsidian MCP Operations Manual

## Toolset Overview

Two MCP servers provide Knowledge Base access:

### basic-memory (`mcp__basic-memory__*`)
- Python-based (`uvx basic-memory`)
- Pointed at Google Drive knowledge base
- Tools: `switch_project`, `build_context`, `list_directory`, `read_note`

### mcp-obsidian (`kb__*`)
- Connects to running Obsidian instance via API
- CRUD: `write_note`, `read_note`, `edit_note`, `delete_note`
- Structure: `list_directory`, `move_note`, `canvas`
- Search: `search_notes`, `recent_activity`, `read_content`
- Multi-project: `create_memory_project`, `switch_project`, `list_memory_projects`

## Key Operational Rules

### Identifier System
Tools use `identifier` (title or path), not raw file paths. Operations are scoped to a "project".

### Anti-patterns (Discovered)
- `create_memory_project` requires absolute paths; relative paths fail
- `folder: "/"` in `write_note` maps to system root (permission errors) -- use named folders
- `read_content` cannot read `memory://` resources -- use `build_context` instead
- Always check existence before `write_note` (fails on existing paths)
- Use `edit_note` for updates (supports `append`, `replace_section`)

### Edit Operations
`edit_note` supports semantic operations:
- `append` — Add to end of note
- `prepend` — Add to beginning
- `find_replace` — Search and replace text
- `replace_section` — Replace a named section

## Relations

- child_of [[Next.js Blog (blogg)]]
- related_to [[Obsidian CMS Pipeline Architecture]]
- related_to [[curator-handover-knowledge]]
