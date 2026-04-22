---
title: Auggie Supergateway Streamable HTTP on Port 7440 (macOS)
type: reference
permalink: operations/macos/auggie-supergateway-streamable-http-on-port-7440-macos
created: 2026-03-13
modified: 2026-03-13
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/auggie
- domain/supergateway
- domain/macos
- domain/mcp
- status/evergreen
---

# Auggie Supergateway Streamable HTTP on Port 7440 (macOS)

## Purpose
Concrete macOS runbook note for exposing Auggie's MCP bootstrap command over local Streamable HTTP on port `7440` using `supergateway`.

## Working Command

```bash
npx -y supergateway \
  --stdio "auggie --mcp --mcp-auto-workspace" \
  --outputTransport streamableHttp \
  --port 7440 \
  --streamableHttpPath /mcp \
  --stateful \
  --healthEndpoint /healthz
```

## Operational Notes

- Launch the command from the repository root you want Auggie to treat as the active workspace.
- If you need a fixed workspace, replace the stdio value with `"auggie --mcp --mcp-auto-workspace -w /absolute/path"`.
- The endpoint exposed to clients is `http://127.0.0.1:7440/mcp`.
- The readiness probe is `http://127.0.0.1:7440/healthz`.
- `--stateful` is recommended so the wrapped stdio process preserves MCP session continuity across requests.
- The local smoke test used `curl -i -s http://127.0.0.1:7440/healthz` and received `200 OK` on 2026-03-13.

## Verification Checklist

1. Start the gateway and wait for `Listening on port 7440`.
2. Check `curl -i -s http://127.0.0.1:7440/healthz` returns `200 OK`.
3. Point the client at `http://127.0.0.1:7440/mcp`.
4. If workspace context is wrong, relaunch from the desired directory or add `-w /absolute/path`.

## Related

- [[Augment Auggie MCP Server Interface]]
- [[Supergateway Auggie Context Engine Deployment Patterns]]
- [[Index – MCP Gateway Operations]]

## Evidence

- Local validation performed on macOS on 2026-03-13.
- Health endpoint observed returning `ok` over HTTP after startup.

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-13 | Created concrete macOS runbook note for Auggie on port 7440. | The user asked for an immediately usable wrapper command and persistence in Basic Memory. | User request to capture validated Auggie wrapping workflow. |

## Sources

- Local validation on 2026-03-13
- `supergateway --help`
- Augment documentation