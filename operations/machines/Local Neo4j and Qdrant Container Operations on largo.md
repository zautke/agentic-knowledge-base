---
title: Local Neo4j and Qdrant Container Operations on largo
type: reference
permalink: kb/operations/machines/local-neo4j-and-qdrant-container-operations-on-largo
entity_type: reference
status: evergreen
created: '2026-03-15'
modified: '2026-03-15'
tags:
- project/agent-kb
- domain/docker
- domain/qdrant
- domain/neo4j
- domain/database
- machine/largo
- status/evergreen
---

# Local Neo4j and Qdrant Container Operations on largo

## Purpose and Scope
Canonical runbook note for the existing local Neo4j and Qdrant containers on `largo`, including live ports, mounts, restart behavior, Codex integration assumptions, and the minimum steps needed to reproduce the same setup on another machine.

## Live Container Inventory
### Qdrant
- Container name: `hybridrag-qdrant`
- Image: `qdrant/qdrant:latest`
- Published ports: `6333` (HTTP), `6334` (gRPC)
- Restart policy: `unless-stopped`
- Bind mount:
  - `/Volumes/MACDEV/indexer/qdrant_storage` -> `/qdrant/storage`
- Observed env:
  - `QDRANT__SERVICE__HTTP_PORT=6333`
  - `QDRANT__SERVICE__GRPC_PORT=6334`
  - `QDRANT__LOG_LEVEL=INFO`

### Neo4j
- Container name: `hybridrag-neo4j`
- Image: `neo4j:5.15-community`
- Published ports: `7474` (HTTP), `7687` (Bolt)
- Restart policy: `unless-stopped`
- Bind mounts:
  - `/Volumes/MACDEV/indexer/neo4j_data` -> `/data`
  - `/Volumes/MACDEV/indexer/neo4j_logs` -> `/logs`
  - `/Volumes/MACDEV/indexer/neo4j_plugins` -> `/plugins`
- Observed bootstrap env in container config:
  - `NEO4J_AUTH=neo4j/hybridrag_secret`
  - `NEO4J_PLUGINS=["graph-data-science"]`
  - `NEO4J_dbms_security_procedures_allowlist=gds.*`
  - `NEO4J_dbms_security_procedures_unrestricted=gds.*`

## Live Data Shape Observed During 2026-03-15 Validation
### Qdrant
- Collection confirmed: `hybridrag_docs`
- Approximate point count observed from collection info: `19`
- Dense vector size observed: `384`

### Neo4j
- Labels confirmed: `Entity`, `Document`, `File`
- Relationship type confirmed: `CALLS`
- Example indexed properties observed: `id`, `path`, `name`, `type`, `signature`, `description`, `embedding`, `filePath`, `lineNumber`, `language`, `pageRankScore`, `weight`

## Codex Integration Notes
- The working local Codex MCP path does not currently rely on the packaged Qdrant or Neo4j MCP servers.
- Codex uses local adapter scripts in `~/.local/bin/` that talk directly to:
  - `http://localhost:6333` for Qdrant
  - `http://localhost:7474` for Neo4j
- Relevant local files:
  - `~/.local/bin/codex_simple_mcp.py`
  - `~/.local/bin/codex_qdrant_mcp_server.py`
  - `~/.local/bin/codex_neo4j_mcp_server.py`
  - `~/.local/bin/codex-qdrant-mcp-bridge`
  - `~/.local/bin/codex-neo4j-mcp-bridge`
  - `~/.codex/config.toml`

## Credential Truth for This Machine
- Qdrant API key from `comp/common_machines/global_env.md` worked during 2026-03-15 validation.
- Neo4j on this machine authenticated with `neo4j / hybridrag_secret` during 2026-03-15 validation.
- The Neo4j password recorded in `global_env` did not match the currently running local container.

## Recreate on Another Machine
### Required assets to preserve
- Qdrant storage directory or equivalent volume contents
- Neo4j data directory
- Neo4j logs directory if forensic continuity matters
- Neo4j plugins directory if GDS or APOC jars must be preserved exactly
- Any environment secrets needed by local MCP wrappers or higher-level apps

### Minimal Qdrant bring-up pattern
```bash
docker run -d \
  --name hybridrag-qdrant \
  --restart unless-stopped \
  -p 6333:6333 -p 6334:6334 \
  -v /absolute/path/qdrant_storage:/qdrant/storage \
  qdrant/qdrant:latest
```

### Minimal Neo4j bring-up pattern
```bash
docker run -d \
  --name hybridrag-neo4j \
  --restart unless-stopped \
  -p 7474:7474 -p 7687:7687 \
  -v /absolute/path/neo4j_data:/data \
  -v /absolute/path/neo4j_logs:/logs \
  -v /absolute/path/neo4j_plugins:/plugins \
  -e NEO4J_AUTH=neo4j/hybridrag_secret \
  -e NEO4J_PLUGINS='["graph-data-science"]' \
  -e NEO4J_dbms_security_procedures_allowlist='gds.*' \
  -e NEO4J_dbms_security_procedures_unrestricted='gds.*' \
  -e NEO4J_server_memory_heap_initial__size=512m \
  -e NEO4J_server_memory_heap_max__size=1G \
  neo4j:5.15-community
```

### Post-start validation on another machine
1. Verify `curl http://localhost:6333/readyz` returns ready.
2. Verify `curl http://localhost:7474` returns Neo4j metadata JSON.
3. Verify Neo4j auth with the expected credential before updating any MCP config.
4. Update local `~/.codex/config.toml` or equivalent client config to point at the recreated endpoints.
5. Run a fresh Codex session and force one tool call against each database adapter.

## Local Risks and Caveats
- The root macOS volume on `largo` was nearly full during the repair session (`249 MiB` free), which interfered with packaged MCP-server/runtime attempts that wanted transient local cache/model files.
- `hybridrag-qdrant` may report unhealthy at the Docker layer while still serving normal HTTP API calls.
- The packaged Neo4j MCP path expected APOC metadata procedures, but the live container configuration only allowlisted `gds.*` and therefore did not satisfy that requirement.
- Future agents should prefer live verification over note-only trust when credentials or plugin state matter.

## Relations
- related_to [[Machine Profile - largo]]
- related_to [[Operational Session - Codex Neo4j and Qdrant MCP Repair on largo (2026-03-15)]]
- related_to [[Retrieval-Augmented Generation (RAG)]]
- related_to [[Fundamental – Agent KB]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-15 | Created container operations note for the existing local Neo4j and Qdrant setup on `largo`. | Future agents needed machine-local database context, exact mount/port details, and a portable bring-up pattern for another machine. | User request to record local machine database context and reproducible spin-up guidance. |