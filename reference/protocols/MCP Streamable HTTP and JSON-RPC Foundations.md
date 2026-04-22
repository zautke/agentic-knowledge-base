---
title: MCP Streamable HTTP and JSON-RPC Foundations
type: reference
permalink: reference/protocols/mcp-streamable-http-and-json-rpc-foundations
created: 2026-03-10
modified: 2026-03-10
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/mcp
- domain/json-rpc
- domain/protocols
- status/evergreen
---

# MCP Streamable HTTP and JSON-RPC Foundations

## Purpose
Operational reference for current MCP transport chronology, JSON-RPC 2.0 response semantics, Streamable HTTP session mechanics, bidirectional request handling, cancellation, progress, and resumability.

## Observations

- [chronology] Standalone HTTP+SSE transport is deprecated as of protocol version `2024-11-05`.
- [chronology] Streamable HTTP is the replacement standard HTTP transport beginning with protocol version `2025-03-26`.
- [chronology] The latest versioned MCP spec material directly verified during this session was `2025-06-18`.
- [protocol] Current standard MCP transports are `stdio` and Streamable HTTP; WebSocket is a custom transport rather than a standard MCP transport.
- [protocol] MCP uses JSON-RPC 2.0 envelopes: a response contains exactly one of `result` or `error`.
- [protocol] In Streamable HTTP, client messages are sent via HTTP `POST`; the server may reply with `application/json` or `text/event-stream`.
- [protocol] If the server returns `Mcp-Session-Id` during initialization, the client must include that header on all subsequent HTTP requests.
- [protocol] `MCP-Protocol-Version` should be sent on subsequent HTTP requests even if some implementations tolerate its absence.
- [protocol] SSE still matters only as optional framing inside Streamable HTTP, not as the current standalone transport.
- [protocol] Cancellation is advisory and asynchronous; disconnect is not equivalent to cancellation.
- [protocol] Progress notifications require an advertised `_meta.progressToken` on the originating request.
- [protocol] Resumability is optional and depends on server support for SSE event ids and `Last-Event-ID` replay.
- [architecture] A serious MCP client is bidirectional: it must handle server-issued requests, not just server responses.
- [operator] `curl` is suitable for atomic protocol experiments but not sufficient as a full MCP client runtime once server-issued requests and replay handling matter.

## Explains
- [[Action Taxonomy – Index]]
- [[Agentic Knowledge Base System Overview]]

## Related
- [[OpenAI Codex MCP Server Interface]]
- [[Manual MCP Client Operation for Codex]]
- [[FastMCP in Agentic Systems]]
- [[LiteLLM in Agentic Systems]]
- [[MCP Capability Boundaries in Agentic Systems]]
- [[Index – Agent Runtime and Capability Stack]]
## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-10 | Created reference note for MCP transport chronology and JSON-RPC/MCP operational semantics. | New operational knowledge cluster did not exist in `agent-kb`. | KB capture of Codex MCP protocol research. |
| 2026-04-17 | Added LiteLLM and FastMCP bridge references. | The MCP foundation note is the closest existing hub for framework-level MCP references and needed outward links to the new global notes. | User request to create global LiteLLM and FastMCP references in broader agentic-systems context. |
| 2026-04-17 | Added MCP boundary and stack-index links. | The MCP foundation note needed to become part of the new broader agent-runtime cluster, not just a protocol-isolated reference. | User request to keep building the global runtime/capability knowledge graph. |
## Sources
- MCP spec transports `2025-03-26`
- MCP spec schema `2025-06-18`
- MCP spec cancellation/progress docs
- JSON-RPC 2.0 specification
- Live validation against local stateful Streamable HTTP deployment