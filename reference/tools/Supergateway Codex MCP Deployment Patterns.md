---
title: Supergateway Codex MCP Deployment Patterns
type: reference
permalink: reference/tools/supergateway-codex-mcp-deployment-patterns
created: 2026-03-10
modified: 2026-03-10
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/supergateway
- domain/codex
- domain/mcp
- status/evergreen
---

# Supergateway Codex MCP Deployment Patterns

## Purpose
Operational reference for wrapping native Codex stdio MCP with Supergateway, including preferred transport choice, stateful session configuration, version pinning, CORS/auth guidance, and operator wrapper strategy.

## Observations

- [transport] For Codex, the preferred Supergateway mode is `stdio -> Streamable HTTP`, not deprecated standalone SSE.
- [transport] `--stateful` is the preferred mode for conversational Codex usage because it aligns with `Mcp-Session-Id` session continuity.
- [transport] WebSocket should be treated as custom compatibility mode rather than the primary MCP transport.
- [version] Latest official Supergateway release verified during this session was `v3.4.3` published on `2025-10-09`.
- [version] Pinning a recent Supergateway version is preferable to floating installs for reproducible operator workflows.
- [ops] Recommended local defaults: port `8080`, path `/mcp`, health endpoint `/healthz`, log level `info`, CORS disabled by default.
- [security] The MCP spec recommends localhost binding, Origin validation, auth, and TLS for HTTP transports; current Supergateway source reviewed during this session did not expose an obvious host-bind flag in the CLI path examined.
- [ops] Add a health endpoint explicitly if automation depends on readiness; otherwise probes like `/healthz` may fail.
- [ops] Wrapper scripts are justified because the canonical Supergateway CLI is verbose and error-prone to type repeatedly.
- [artifact] Current repo wrapper script: `/Volumes/FLOUNDER/dev/wxt-prompt/scripts/codex-supergateway.sh`.

## Depends on
- [[MCP Streamable HTTP and JSON-RPC Foundations]]
- [[OpenAI Codex MCP Server Interface]]

## Related
- [[Manual MCP Client Operation for Codex]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-10 | Created Supergateway deployment reference note. | Needed a distinct operational note for gateway configuration and wrapper guidance. | KB capture of Supergateway research and repo script/manual additions. |

## Sources
- Supergateway README
- Supergateway latest release metadata
- Supergateway stateful Streamable HTTP source
- MCP transport security and session guidance