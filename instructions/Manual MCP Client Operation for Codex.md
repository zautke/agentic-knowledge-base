---
title: Manual MCP Client Operation for Codex
type: instruction
permalink: instructions/manual-mcp-client-operation-for-codex
created: 2026-03-10
modified: 2026-03-10
entity_type: instruction
status: draft
tags:
- project/agent-kb
- domain/codex
- domain/mcp
- domain/protocols
- op/create-node
- status/draft
- source/largo
- machine/largo
---

# Manual MCP Client Operation for Codex

## Purpose
Operational instruction for manually interacting with a wrapped Codex MCP server over Streamable HTTP, including initialization, session capture, tool discovery, tool invocation, bidirectional client obligations, and context handoff.

## Procedure Summary

1. Send `initialize` to the MCP endpoint.
2. Capture `Mcp-Session-Id` from the HTTP response header.
3. Send `notifications/initialized` with the session id.
4. Call `tools/list`.
5. Call `tools/call` with `name = "codex"` to create a thread.
6. Extract `structuredContent.threadId`.
7. Call `tools/call` with `name = "codex-reply"` to continue the thread.
8. Treat any server-issued JSON-RPC request as requiring a JSON-RPC response from the client.
9. Treat `curl` as an atomic inspection tool, not a complete long-lived MCP client runtime.

## Critical Constraints

- Include `Mcp-Session-Id` after initialization if the server returned it.
- Prefer sending `MCP-Protocol-Version` on subsequent HTTP requests.
- Distinguish protocol errors from tool execution errors.
- Distinguish `threadId` from `Mcp-Session-Id`.
- Standalone SSE is deprecated; in current practice it appears only as optional framing within Streamable HTTP.
- WebSocket use in this domain is a custom gateway transport, not a standard MCP transport guarantee.

## Related
- [[MCP Streamable HTTP and JSON-RPC Foundations]]
- [[OpenAI Codex MCP Server Interface]]
- [[Supergateway Codex MCP Deployment Patterns]]
- [[Index – MCP and Codex Operations]]
## Evidence
- Canonical working manual: `/Volumes/FLOUNDER/dev/wxt-prompt/docs/codex-mcp-operations-manual.md`
- Supergateway wrapper script: `/Volumes/FLOUNDER/dev/wxt-prompt/scripts/codex-supergateway.sh`
## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-10 | Created operational instruction node pointing to the repo manual as SSOT for copy-pastable command sequences. | Needed executable operational guidance without duplicating the full repo manual into KB. | KB capture of Codex MCP manual work. |

| 2026-03-10 | Linked Supergateway deployment reference and wrapper script artifact. | The operational instruction now depends on gateway configuration guidance and a concrete wrapper entrypoint. | KB capture of Supergateway research and repo script addition. |