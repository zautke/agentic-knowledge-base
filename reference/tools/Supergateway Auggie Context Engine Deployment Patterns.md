---
title: Supergateway Auggie Context Engine Deployment Patterns
type: reference
permalink: reference/tools/supergateway-auggie-context-engine-deployment-patterns
created: 2026-03-13
modified: 2026-03-13
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/supergateway
- domain/auggie
- domain/mcp
- status/evergreen
---

# Supergateway Auggie Context Engine Deployment Patterns

## Purpose
Operational reference for wrapping Augment's Auggie context engine in `supergateway` so it is exposed over Streamable HTTP, with emphasis on stateful session handling, endpoint layout, port `7440`, and local smoke-test results.

## Observations

- The preferred wrapping mode for Auggie is `stdio -> Streamable HTTP`.
- `--stateful` is the safer default for Auggie over Streamable HTTP because the wrapped stdio server needs per-client session continuity once initialized.
- The validated local command on 2026-03-13 was:
  `npx -y supergateway --stdio "auggie --mcp --mcp-auto-workspace" --outputTransport streamableHttp --port 7440 --streamableHttpPath /mcp --stateful --healthEndpoint /healthz`
- The resulting MCP endpoint is `http://127.0.0.1:7440/mcp`.
- The explicit health endpoint is `http://127.0.0.1:7440/healthz`.
- Local smoke test on 2026-03-13 returned `HTTP/1.1 200 OK` for `/healthz` after Supergateway reported `Listening on port 7440`.
- `supergateway --help` on this machine documented `--outputTransport streamableHttp`, `--streamableHttpPath`, `--stateful`, and `--healthEndpoint`.
- CORS is disabled by default; only enable it when a browser-origin client actually requires it.
- If the operator needs a fixed workspace independent of launch directory, embed `-w /absolute/path` in the `--stdio` command string.

## Recommended Defaults

- Port: `7440`
- Streamable HTTP path: `/mcp`
- Health endpoint: `/healthz`
- Output transport: `streamableHttp`
- Session mode: `--stateful`
- Host assumption: localhost-only unless a separate reverse proxy/tunnel is added deliberately

## Related

- [[Augment Auggie MCP Server Interface]]
- [[MCP Streamable HTTP and JSON-RPC Foundations]]
- [[Auggie Supergateway Streamable HTTP on Port 7440 (macOS)]]
- [[Index – MCP Gateway Operations]]
- [[Supergateway Codex MCP Deployment Patterns]]

## Evidence

- Local command validation on 2026-03-13: `npx -y supergateway --help`
- Local smoke test on 2026-03-13: gateway startup + `curl -i -s http://127.0.0.1:7440/healthz`
- MCP transport and Augment docs verified live during the session.

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-13 | Created Supergateway deployment reference for Auggie context-engine wrapping. | Existing KB note covered Codex wrapping only; Auggie needed its own validated deployment pattern. | User request to capture Auggie + Supergateway operational context in Basic Memory. |

## Sources

- Local `supergateway` help and smoke test on 2026-03-13
- MCP transport specification
- Augment documentation