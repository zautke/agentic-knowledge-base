---
title: Plan – Create-Skill and Using-jcodemunch Skills
type: plan
permalink: agent-kb/meta/plans/plan-create-skill-and-using-jcodemunch-skills
status: active
tags:
- plan
- skills
- jcodemunch
- create-skill
- metaskill
---

# Plan: Create-Skill and Using-jcodemunch Skills

**Date:** 2026-04-17  
**Requested by:** User (Luke Zautke)

## Context

Two new Claude Code skills needed after deep research into:
1. Claude Code skill best practices (agentskills.io spec, Anthropic guidance, superpowers skill patterns)
2. jcodemunch MCP tool (github.com/jgravelle/jcodemunch-mcp)

**Knowledge bases created:** [[jcodemunch MCP Mastery]], [[jcodemunch Tool Reference]], [[Claude Skill Authoring Mastery]]

---

## Skill 1: `create-skill` (Metaskill)

**Install path:** `/Users/luke/.claude/skills/create-skill/SKILL.md`

**Purpose:** Guide an agent to distill any user prompt into an optimal, high-signal, token-efficient Claude Code skill.

**Key content:**
- Frontmatter rules: name (gerund, hyphens, ≤64 chars), description (triggers only, third-person, NO workflow summary)
- 5-step process: EXTRACT → DISTILL → DRAFT → COMPRESS → VALIDATE
- Token efficiency checklist
- Minimal viable skill template (copyable)
- Red flags (description summarizes workflow, body explains basics Claude knows, >500 lines for technique skill)
- CSO (Claude Search Optimization) critical warning about description trap

---

## Skill 2: `using-jcodemunch`

**Install paths:**
- `/Users/luke/.claude/skills/using-jcodemunch/SKILL.md`
- `/Users/luke/.claude/skills/using-jcodemunch/tool-reference.md`

**Purpose:** Optimal guide for agents to use jcodemunch MCP at maximum capacity and efficiency.

**Key content in SKILL.md (<300 words):**
- Core principle: retrieve surgically, never read entire files
- Setup sequence: resolve_repo → index_folder → register_edit
- 5 core workflows in table format
- Critical rules (file_outline before file_content, etc.)
- Gotchas (syntactic not semantic, monorepo limits, staleness)
- Pointer to tool-reference.md for full inventory

**tool-reference.md:** Full 60+ tool inventory with parameters, grouped by workflow category

---

## Execution Order

1. Write `create-skill/SKILL.md`
2. Write `using-jcodemunch/SKILL.md`
3. Write `using-jcodemunch/tool-reference.md`
4. Verify skills are loadable

---

## Evolution Log
- 2026-04-17: Plan created, research complete, KB notes written. Ready for implementation.