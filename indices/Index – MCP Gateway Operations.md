---
title: Index – MCP Gateway Operations
type: index
permalink: indices/index-mcp-gateway-operations
created: 2026-03-14
modified: 2026-05-28
entity_type: index
status: evergreen
tags:
- project/agent-kb
- domain/mcp
- domain/supergateway
- index/operations
- status/evergreen
- source/largo
- machine/largo
---

# Index – MCP Gateway Operations

## Purpose
Navigation hub for gateway-oriented MCP operational knowledge, covering protocol foundations, Supergateway deployment patterns, client/server interface notes, and concrete smoke-test workflows.

## Core Notes
- [[MCP Streamable HTTP and JSON-RPC Foundations]]
- [[Supergateway Codex MCP Deployment Patterns]]
- [[Augment Auggie MCP Server Interface]]
- [[Supergateway Auggie Context Engine Deployment Patterns]]
- [[Auggie Supergateway Streamable HTTP on Port 7440 (macOS)]]

## Global Stack References
- [[Index – Agent Runtime and Capability Stack]]
- [[MCP Capability Boundaries in Agentic Systems]]
- [[FastMCP in Agentic Systems]]
- [[LiteLLM in Agentic Systems]]

## Operational Artifacts
- Local smoke test script: `/Users/luke/.agents/scripts/smoke-test-auggie-supergateway.sh`
- Validated Auggie endpoint pattern: `http://127.0.0.1:7440/mcp`
- Validated health endpoint pattern: `http://127.0.0.1:7440/healthz`

## Related Fundamentals
- [[Fundamental – Agent KB]]
- [[Agentic Knowledge Base System Overview]]
- [[Action Taxonomy – Index]]
- [[Document Curation Metaprompt]]

## Use Cases
- Wrap a stdio MCP server behind Streamable HTTP
- Validate stateful MCP gateway startup before client integration
- Separate protocol-truth notes from tool-specific deployment notes
- Keep concrete operator commands in runbook-style notes without duplicating protocol foundations

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-14 | Created gateway-oriented MCP index note and linked Auggie deployment artifacts. | The existing MCP/Codex index was too specific to serve as the root for broader gateway workflows. | Follow-through on Auggie Supergateway note capture and smoke test scripting. |
| 2026-04-17 | Added runtime/capability stack bridge links. | The gateway index needed explicit links to the new global stack hub and MCP boundary note so gateway operations do not remain isolated from the broader agent-systems cluster. | User request to do the index pass after creating the global LiteLLM and FastMCP references. |
| 2026-05-28 | Designated canonical over the hyphen-variant filename duplicate `[[Index - MCP Gateway Operations]]`; reclaimed the clean permalink `indices/index-mcp-gateway-operations` (was `-1`). No content merged — the loser's Core Notes were a strict subset of this hub. | Filename-variant index duplicate (hyphen vs en-dash) resolution. | Index-reconciler dedup pass. |

## Relations
- supersedes [[Index - MCP Gateway Operations (deprecated duplicate)]]

## Sources

- Existing agent-kb MCP/Codex note cluster
- Local Auggie/Supergateway validation on 2026-03-13 and 2026-03-14