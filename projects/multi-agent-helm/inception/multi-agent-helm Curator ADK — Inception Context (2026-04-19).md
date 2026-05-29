---
title: multi-agent-helm Curator ADK — Inception Context (2026-04-19)
type: context
entity_type: context
status: active
created: 2026-04-19
modified: 2026-05-28
permalink: kb/projects/multi-agent-helm/inception/multi-agent-helm-curator-adk-inception-context-2026-04-19
project: agent-kb
cross_project: multi-agent-helm
date: 2026-04-19
tags:
- project/multi-agent-helm
- agent/curator-adk
- domain/agentic-systems
- domain/curation
- status/inception
- phase/1
- cross-project-pointer
---

# multi-agent-helm Curator ADK — Inception Context (2026-04-19)

**Cross-project pointer.** This note exists in the global `kb` project so that future searches across the agentic KB surface the `multi-agent-helm` Curator ADK sub-project. The authoritative entity notes live in the dedicated repo-local Basic-Memory project rooted at `/Volumes/MACDEV/multi_agent_helm/.basic-memory`. (The repository directory is named `multi_agent_helm` on disk; the canonical KB project slug is `multi-agent-helm`.)

## Why this note exists

The user is building the Curator ADK Agent — a Google ADK-based standalone service that exposes the existing Curator persona (`~/.claude/agents/curator.md`) over A2A (canonical), MCP (Streamable HTTP façade for hosts), and optional ACP (legacy compat). It becomes a first-class persistent team member in Claude Code, OpenAI Codex CLI, and OpenCode. It lives at `/Volumes/MACDEV/multi_agent_helm/agents/curator-adk/`.

The `multi-agent-helm` repository follows a dual-KB persistence pattern (precedent: `graphrag-off-main-kb`):

- Repo-local Basic-Memory project (`multi_agent_helm`) at `/Volumes/MACDEV/multi_agent_helm/.basic-memory` for entity-level inception, architecture, and decision notes that travel with the codebase.
- This single context note in the global `kb` project for cross-project discoverability.

## Where to find everything

### Repo-local KB notes (project: `multi_agent_helm`)

- `[[Inception/00 Charter]]` (`inception/00-charter-multi-agent-helm-curator-adk-inception`) — why the project exists, dual-KB rationale, locked decisions, open questions
- `[[Inception/01 Architecture Snapshot]]` (`inception/01-architecture-snapshot-curator-adk-service-topology`) — system diagram, protocol surface, memory layering, decision log
- `[[Inception/02 Curator ADK Summary]]` (`inception/02-curator-adk-summary-repo-doc-pointer`) — pointer to in-repo Markdown docs and planned file layout

To browse the repo-local project: `list_directory("/inception", project="multi_agent_helm")`.

### In-repo Markdown docs (canonical source-of-truth)

- `/Volumes/MACDEV/multi_agent_helm/agents/curator-adk/PRD.md` — Product requirements
- `/Volumes/MACDEV/multi_agent_helm/agents/curator-adk/PLANNING.md` — Architecture and planning
- `/Volumes/MACDEV/multi_agent_helm/agents/curator-adk/TASKS.md` — 26-task implementation ledger
- `/Volumes/MACDEV/multi_agent_helm/agents/SESSION_HISTORY.md` — Cross-agent session log (Session 001 = this inception)
- `/Volumes/MACDEV/multi_agent_helm/agents/curator-adk/__init__.py` — empty placeholder; modules added in Phase 2

### Helm context (already in repo)

- `/Volumes/MACDEV/multi_agent_helm/README.md` — Kubernetes-deployed Kafka + pgEdge + Redis ecosystem
- `/Volumes/MACDEV/multi_agent_helm/ARCHITECTURE.md` — 1060-line LangGraph supervisor spec with 4 personas (Knuth/Overseer/Curmudgeon/Scribe). The new ADK service generalizes The Scribe.
- `/Volumes/MACDEV/multi_agent_helm/distributed/` — uv workspace with services + packages, doc precedent for our `TASKS.md` and `SESSION_HISTORY.md`

### Inheritance source

- `~/.claude/agents/curator.md` — the local Claude Code-only Curator subagent the new ADK service generalizes. Its 7-gate scoring rubric (Gate 0 Authorization 10, Gate 1 Classification 10, Gate 2 Dedup 10, Gate 3 Composition 20, Gate 4 Linking 25, Gate 5 Verification 10, Gate 6 Oversight 15 = 100; pass at ≥91) and telemetry log pattern are inherited verbatim.

## Decisions locked in this session (2026-04-19)

- ADK ≥ 1.31 with `[a2a]` extra is the agent core framework.
- A2A spec 1.0 is canonical; ACP legacy adapter behind `--enable-acp` (ACP merged into A2A Aug 2025).
- MCP Streamable HTTP façade exposes 4 high-level Curator tools to host agents: `submit_for_review`, `query_memory`, `recent_decisions`, `bootstrap`.
- Memory: Basic-Memory MCP (markdown SoR) + pluggable BaseMemoryService recall (Mem0 local / Vertex Memory Bank cloud / InMemory test).
- Sessions: ADK `DatabaseSessionService` (SQLite local / Postgres cloud).
- Deployment: launchd local daemon (port 8765, kill-stale rule), Cloud Run, Vertex Agent Engine.
- Three-host integration as `curator-remote`: Claude Code, Codex CLI, OpenCode.
- Observability: OpenTelemetry → Langfuse + Phoenix; Cloud Trace on GCP.
- Eval: `adk eval` with `rubric_based_final_response_quality_v1` encoding the 7-gate rubric.
- Conformance: `a2a-tck` (mandatory tier) + `@modelcontextprotocol/conformance`.
- Coexists with existing `~/.claude/agents/curator.md` during rollout.
- Dual-KB pattern follows `graphrag-off-main-kb` precedent.

## What's open

- `pyproject.toml` layout: own vs join helm-root uv workspace (`distributed/pyproject.toml` already declares one for `services/*` + `packages/*`). Documented in `agents/curator-adk/PLANNING.md` Open Questions.
- `.basic-memory/` git tracking: helm has no `.gitignore` at root today. Default: track in git so KB travels with code.
- Deprecation timing for `~/.claude/agents/curator.md` after `curator-remote` field-tests green.
- Subagent model slug: this session ran with `composer-2` (only model exposed in this environment); user requested Sonnet — confirm exact slug for next session.

## Phase status

- **Phase 1 (this session)** — DONE: docs authored (PRD, PLANNING, TASKS, SESSION_HISTORY), package layout pinned (`__init__.py`), dual KB inception complete.
- **Phase 2 (next session)** — PENDING: execute the 26 tasks in `agents/curator-adk/TASKS.md`.

## Relations

- part_of [[Project Index]]
- mirrors [[Inception/00 Charter]] (multi_agent_helm project)
- mirrors [[Inception/01 Architecture Snapshot]] (multi_agent_helm project)
- mirrors [[Inception/02 Curator ADK Summary]] (multi_agent_helm project)
- generalizes [[Curator]] (`~/.claude/agents/curator.md`)
- governed_by [[KB Submission Quality Protocol]]
- governed_by [[Curator Handover Knowledge]]
- references [[Fundamental – Agent KB]]

## References

- Repo: `/Volumes/MACDEV/multi_agent_helm/`
- Repo-local KB: project `multi_agent_helm` at `/Volumes/MACDEV/multi_agent_helm/.basic-memory`
- A2A spec 1.0: `https://a2a-protocol.org/v1.0.0/specification`
- ADK docs: `https://google.github.io/adk-docs/`
- ACP merger announcement: `https://lfaidata.foundation/communityblog/2025/08/29/acp-joins-forces-with-a2a-under-the-linux-foundations-lf-ai-data/`

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-05-28 | Relocated from `projects/multi_agent_helm/inception/` to canonical hyphen-slug folder `projects/multi-agent-helm/inception/`; normalized slug, tags (`project/multi_agent_helm`→`project/multi-agent-helm`), added missing frontmatter (entity_type, status, created, modified), resolved relation targets to existing notes, linked into project hub. | Slug-variant deduplication (underscore vs hyphen); the underscore folder was a non-canonical duplicate of the hyphen project. | KB curation sweep — projects/ partition. |
