---
title: Augment Auggie MCP Server Interface
type: reference
permalink: reference/tools/augment-auggie-mcp-server-interface
created: 2026-03-13
modified: 2026-03-13
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/augment
- domain/auggie
- domain/mcp
- status/evergreen
---

# Augment Auggie MCP Server Interface

## Purpose
Operational reference for Augment's `auggie` CLI when used as an MCP server bootstrap, with emphasis on the hidden `--mcp` and `--mcp-auto-workspace` flags, workspace-root behavior, and local verification of the installed CLI surface.

## Observations

- On this machine, `auggie` is installed at `/Users/luke/.config/nvm/versions/node/v24.13.0/bin/auggie`.
- Local CLI help on 2026-03-13 did not print `--mcp` or `--mcp-auto-workspace` in the public top-level help output, which implies these flags should be treated as supported-but-underdocumented operator flags rather than relying on local `--help` discovery alone.
- Augment's live CLI reference documented the bootstrap command `auggie --mcp --mcp-auto-workspace` during this session.
- `--mcp` is the bootstrap mode that exposes Auggie as an MCP server process over stdio.
- `--mcp-auto-workspace` makes the process infer workspace context from the current working directory unless an explicit workspace root is also supplied.
- `-w, --workspace-root <path>` remains relevant when the operator wants a stable primary workspace instead of pure current-directory inference.
- For gateway wrapping, the stdio command should be treated as the canonical bootstrap string and preserved verbatim inside the gateway configuration.

## Critical Constraints

- Start the gateway from the repository root you want Auggie to treat as the active workspace, or pass `-w /absolute/path` explicitly inside the wrapped command.
- Do not assume the local `auggie --help` output is complete for MCP-related flags; verify against Augment docs when operator behavior matters.
- `auggie mcp` is a configuration-management subcommand, not the same thing as running `auggie --mcp` as a stdio MCP server.

## Related

- [[MCP Streamable HTTP and JSON-RPC Foundations]]
- [[Supergateway Auggie Context Engine Deployment Patterns]]
- [[Auggie Supergateway Streamable HTTP on Port 7440 (macOS)]]
- [[Index – MCP Gateway Operations]]

## Evidence

- Local command validation on 2026-03-13: `command -v auggie`, `auggie --help`, `auggie mcp --help`
- Augment CLI reference and integrations docs verified live during the session.

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-13 | Created Auggie MCP server interface reference note. | The KB had Supergateway/Codex material but no Auggie-specific MCP bootstrap note. | User request to capture current Auggie MCP wrapping knowledge in Basic Memory. |

## Sources

- Local CLI inspection on 2026-03-13
- Augment CLI reference
- Augment CLI integrations documentation