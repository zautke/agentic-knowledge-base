---
title: Operational Session - Codex Neo4j and Qdrant MCP Repair on largo (2026-03-15)
type: reference
permalink: kb/operations/machines/operational-session-codex-neo4j-and-qdrant-mcp-repair-on-largo-2026-03-15
entity_type: reference
status: draft
created: '2026-03-15'
modified: '2026-03-15'
tags:
- project/agent-kb
- domain/mcp
- domain/codex
- domain/qdrant
- domain/neo4j
- domain/docker
- machine/largo
- status/draft
---

# Operational Session - Codex Neo4j and Qdrant MCP Repair on largo (2026-03-15)

## Purpose and Scope
Capture the full local repair session used to restore usable `qdrant` and `neo4j` MCP entries in `~/.codex/config.toml` on `largo`, including discovery sources, runtime findings, and the final working adapter approach.

## Session Trigger
The local Codex configuration contained `qdrant` and `neo4j` MCP entries, but both failed during Codex MCP startup with `connection closed: initialize response`, so the tools were not available in live sessions.

## Discovery Methods Used
- Obsidian search to locate `comp/common_machines/global_env.md`
- local shell inspection of `~/.codex/config.toml`
- Docker inspection of `hybridrag-qdrant` and `hybridrag-neo4j`
- local HTTP probes against Qdrant and Neo4j
- GitHub and web research against official Qdrant, Neo4j MCP, and `mcp-remote` documentation
- Context7 queries for `mcp-remote` and Neo4j MCP server documentation
- Basic Memory KB review of [[Fundamental – Agent KB]] and [[Agentic Knowledge Base System Overview]] before writing KB outputs

## Starting State
### Qdrant
- Existing database container: `hybridrag-qdrant`
- Live endpoint: `http://localhost:6333`
- API key from `global_env` worked against the live service.
- `GET /collections` returned collection `hybridrag_docs`.

### Neo4j
- Existing database container: `hybridrag-neo4j`
- Live endpoints: `http://localhost:7474` and `bolt://localhost:7687`
- The password stored in `global_env` did not authenticate to the currently running container.
- The live container accepted `neo4j / hybridrag_secret` and rejected the `global_env` password.

## What Failed
### Official Qdrant MCP server path
- The packaged `mcp-server-qdrant` path did not become usable in this environment.
- Deep inspection showed the server attempted to load the FastEmbed ONNX model from a broken cache path under `/var/folders/...` on the nearly-full root volume.
- Root free space during the session was about `249 MiB`, which is consistent with the broken local cache state.

### Official Neo4j MCP server path
- The preexisting `mcp/neo4j:latest` path initially failed because the configured password was wrong for the live container.
- After switching to the live credential, the packaged server progressed further but then failed with `please ensure the APOC plugin is installed and includes the 'meta' component`.
- The live Neo4j container only allowlisted `gds.*` through environment-driven config, so APOC procedures required by that packaged MCP path were still blocked.

## Runtime Findings
- `hybridrag-qdrant` was unhealthy by Docker health status but served requests successfully.
- `hybridrag-neo4j` used bind mounts under `/Volumes/MACDEV/indexer/` and exposed a working transactional HTTP endpoint.
- Qdrant collection state confirmed a dense vector size of `384` for `hybridrag_docs`.
- Neo4j schema overview confirmed labels `Entity`, `Document`, and `File`, with relationship type `CALLS`.

## Repair Implemented
### Final approach
Instead of continuing to wrap failing packaged MCP servers, the repair replaced both bridges with tiny direct stdio MCP adapters that talk straight to the already-running local services:
- `codex-qdrant-mcp-bridge` now launches a local Python MCP adapter for Qdrant HTTP APIs.
- `codex-neo4j-mcp-bridge` now launches a local Python MCP adapter for Neo4j's HTTP transactional API.

### Files created or updated locally
- `~/.codex/config.toml`
- `~/.local/bin/codex_simple_mcp.py`
- `~/.local/bin/codex_qdrant_mcp_server.py`
- `~/.local/bin/codex_neo4j_mcp_server.py`
- `~/.local/bin/codex-qdrant-mcp-bridge`
- `~/.local/bin/codex-neo4j-mcp-bridge`

### Config truth after repair
- `qdrant` uses `QDRANT_URL=http://localhost:6333`, the live API key, and default collection `hybridrag_docs`.
- `neo4j` uses `NEO4J_HTTP_URL=http://localhost:7474`, `NEO4J_USERNAME=neo4j`, and `NEO4J_PASSWORD=hybridrag_secret`.

## Validation
### Direct adapter validation
- Manual MCP initialize and `tools/list` succeeded for both adapters.
- Manual tool calls succeeded for `qdrant_list_collections` and `neo4j_db_overview` when launched with the same env that Codex provides.

### Codex validation
A fresh `codex exec` session on 2026-03-15 reported:
- `mcp: neo4j ready`
- `mcp: qdrant ready`

The same session successfully called:
- `qdrant.qdrant_list_collections()` -> returned `hybridrag_docs`
- `neo4j.neo4j_db_overview()` -> returned labels `Entity`, `Document`, `File` and relationship type `CALLS`

## Important Caveat
`global_env` was a valid source of truth for the Qdrant endpoint and API key in this session, but it was not the source of truth for the currently running Neo4j password on `largo`. Future agents should verify live Neo4j auth against the container before blindly trusting that note.

## Sources
- `comp/common_machines/global_env.md`
- official Qdrant MCP repo documentation
- official Neo4j MCP documentation and repo docs
- `mcp-remote` documentation
- local Docker inspect/log output
- local HTTP verification against `localhost:6333` and `localhost:7474`

## Relations
- related_to [[Machine Profile - largo]]
- related_to [[Index – MCP and Codex Operations]]
- related_to [[MCP Streamable HTTP and JSON-RPC Foundations]]
- related_to [[OpenAI Codex MCP Server Interface]]
- related_to [[Local Neo4j and Qdrant Container Operations on largo]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-15 | Created operational repair log for the local Codex Neo4j/Qdrant MCP recovery on `largo`. | Preserve the exact runtime findings and the reason a direct adapter approach was chosen over the packaged MCP servers. | User request to repair the entries, validate them in Codex, and record the experience in the KB. |