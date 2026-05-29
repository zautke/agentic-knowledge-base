---
title: MCP Capability Boundaries in Agentic Systems
type: reference
permalink: agent-kb/reference/protocols/mcp-capability-boundaries-in-agentic-systems
created: 2026-04-21
modified: 2026-05-28
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/mcp
- domain/agent-systems
- domain/architecture
- status/evergreen
---

# MCP Capability Boundaries in Agentic Systems

## Purpose
Global reference for deciding what should live at an MCP boundary versus what should remain in a higher-level agent runtime.

## Core Position
MCP is best used as a **capability boundary**, not as a synonym for the entire agent system.

It is excellent for exposing tools, resources, prompts, and typed capability surfaces across process or network boundaries. It is weaker as the sole owner of application-specific memory, thread state, workflow orchestration, or UI reduction logic.

## Observations

- [boundary] MCP is strongest when it separates privileged or reusable capabilities from the runtime that decides when and why to invoke them.
- [boundary] Tools, resources, prompts, and typed schemas are natural MCP responsibilities.
- [runtime] Thread identity, memory ownership, turn reduction, interruption semantics, and workflow checkpoints are usually runtime responsibilities above MCP.
- [client-duty] Serious MCP clients remain responsible for connection lifecycle, initialization, server-issued requests, and policy decisions around approvals and orchestration.
- [sampling] MCP sampling expands what can happen inside the boundary, but it does not erase the distinction between a capability server and a full workflow engine.
- [multi-agent] In multi-agent systems, MCP is useful as a shared capability fabric, but agent-to-agent coordination still needs additional runtime or workflow structure above raw tool access.
- [transport] STDIO is a strong local boundary for client-launched tools; Streamable HTTP is a strong shared boundary for remote capability services.
- [security] Authentication, approval policy, and side-effect control become sharper and more auditable when tool execution is pushed behind an MCP boundary.
- [anti-pattern] Treating MCP as if it automatically provides durable memory, planning, or graph orchestration creates architectural confusion and leaky responsibility lines.
- [stack] LiteLLM and FastMCP can both participate in an MCP-aware stack, but they occupy different roles: LiteLLM is a gateway/provider layer, while FastMCP is a capability framework.

## Boundary Decision Heuristic

### Put it behind MCP when
- multiple runtimes or agents need the same capability
- the capability is privileged, operational, or side-effectful
- you want schema-first access and transport portability
- you want a stable client/server seam independent of the main app runtime

### Keep it above MCP when
- the logic owns user conversation state or thread memory
- the logic decides long-lived workflow transitions
- the logic coordinates multiple tools and sub-agents as part of one application-specific runtime
- the logic reduces events into UI state or database rows specific to one product

## Relationship To Other Layers

### Runtime layer
Examples: LangGraph-style workflows, app-specific turn engines, durable memory/checkpoint systems.

### Gateway layer
Examples: LiteLLM router/proxy, auth, logging, budget, provider normalization.

### Capability layer
Examples: FastMCP servers and clients, MCP tool/resource/prompt surfaces.

These layers can interact closely, but they should not be conflated.

## Related
- [[MCP Streamable HTTP and JSON-RPC Foundations]]
- [[FastMCP in Agentic Systems]]
- [[LiteLLM in Agentic Systems]]
- [[Index – Agent Runtime and Capability Stack]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-17 | Created MCP capability-boundary reference note. | The KB needed a reusable note explaining what MCP should own versus what should stay in runtimes and orchestration layers. | User request to turn current LiteLLM and FastMCP research into a broader agent-systems knowledge cluster. |

## Sources
- MCP spec operational truths already captured in `MCP Streamable HTTP and JSON-RPC Foundations`
- FastMCP official docs reviewed 2026-04-17
- LiteLLM official docs reviewed 2026-04-17
- LangGraph architectural framing referenced in current `wxt-prompt` planning research