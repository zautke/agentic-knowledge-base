---
title: OpenAI Codex MCP Server Interface
type: reference
permalink: reference/tools/open-ai-codex-mcp-server-interface
created: 2026-03-10
modified: 2026-03-10
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/codex
- domain/mcp
- domain/tools
- status/evergreen
---

# OpenAI Codex MCP Server Interface

## Purpose
Operational reference for the Codex MCP server surface, including native transport assumptions, exposed MCP tools, approval-cycle implications, and distinction between session identity and conversation identity.

## Observations

- [transport] Official Codex MCP documentation describes the native Codex MCP server transport as stdio MCP.
- [architecture] Streamable HTTP or WebSocket exposure for Codex is a wrapper/gateway concern, not a native Codex server guarantee.
- [tools] The live wrapped Codex MCP endpoint validated during this session exposed two MCP tools: `codex` and `codex-reply`.
- [tools] `codex` starts a new Codex conversation and returns a `threadId` plus content.
- [tools] `codex-reply` continues an existing Codex conversation using `threadId`; `conversationId` is retained only as backward compatibility.
- [identity] `Mcp-Session-Id` identifies the HTTP transport session; `threadId` identifies the Codex conversation. They must not be conflated.
- [approvals] Current Codex MCP documentation describes server-issued approval requests such as `applyPatchApproval` and `execCommandApproval` that require client responses.
- [client-duty] A client supervising Codex must be able to answer those server-issued requests, not just invoke `tools/call`.
- [handoff] Thread continuity is the highest-fidelity handoff primitive for Codex peers; exact live context-window cloning is not a valid design assumption.

## Depends on
- [[MCP Streamable HTTP and JSON-RPC Foundations]]

## Related
- [[Manual MCP Client Operation for Codex]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-10 | Created Codex-specific MCP reference note. | Needed a Codex overlay distinct from general MCP transport/protocol truth. | KB capture of Codex MCP research and live endpoint validation. |

## Sources
- OpenAI Codex MCP interface note
- Live `tools/list` validation against wrapped Codex MCP endpoint
- Repo manual `docs/codex-mcp-operations-manual.md`