---
title: system-architecture-diagram
type: note
entity_type: note
status: active
created: 2026-04-11
modified: 2026-05-28
permalink: kb/projects/multi-agent-helm/system-architecture-diagram
tags:
- project/multi-agent-helm
- domain/multi-agent
- domain/kubernetes
- domain/architecture
- artifact/diagram
- status/active
---

# System Architecture Diagram

This note stores the Mermaid system architecture diagram for the `distributed/` multi-agent Helm scaffold.

```mermaid
flowchart LR
    User[Developer or Client] -->|task up / HTTP clients| Entry[Taskfile + PowerShell Scripts]
    Entry -->|build images| Images[Local Docker Images]
    Entry -->|create cluster + helmfile sync| K3d[k3d Cluster]
    Images --> K3d

    subgraph Cluster["Kubernetes Namespace: a2a-platform"]
        Kong[Kong DB-less Gateway]

        subgraph Agents["FastAPI Agent Runtime Deployments"]
            Supervisor[supervisor\nLangGraph supervisor runtime]
            Agent1[agent-1\nworker runtime]
            Agent2[agent-2\nworker runtime]
            Agent3[agent-3\nworker runtime]
            Agent4[agent-4\nworker runtime]
        end

        Broker[Python gRPC Broker]
        NATS[NATS JetStream]
        Redis[(Redis 8\nRedisSaver + RedisVL + Mem0 backend)]
        Postgres[(PostgreSQL 18\nPostGIS + pg_cron + postgres_fdw)]
        Chroma[(ChromaDB)]
        Config[Platform Config + Secrets]
    end

    User -->|JSON-RPC / HTTP / SSE / WS| Kong
    Kong --> Supervisor
    Kong --> Agent1
    Kong --> Agent2
    Kong --> Agent3
    Kong --> Agent4

    Supervisor <-->|gRPC publish / request / subscribe| Broker
    Agent1 <-->|gRPC publish / request / subscribe| Broker
    Agent2 <-->|gRPC publish / request / subscribe| Broker
    Agent3 <-->|gRPC publish / request / subscribe| Broker
    Agent4 <-->|gRPC publish / request / subscribe| Broker

    Broker <-->|subjects + durable streams| NATS

    Supervisor -->|checkpointing / memory| Redis
    Agent1 -->|shared memory / vector search| Redis
    Agent2 -->|shared memory / vector search| Redis
    Agent3 -->|shared memory / vector search| Redis
    Agent4 -->|shared memory / vector search| Redis

    Supervisor -->|task state / relational data / cron| Postgres
    Agent1 -->|documents / retrieval| Chroma
    Agent2 -->|documents / retrieval| Chroma
    Agent3 -->|documents / retrieval| Chroma
    Agent4 -->|documents / retrieval| Chroma

    Config --> Kong
    Config --> Broker
    Config --> Supervisor
    Config --> Agent1
    Config --> Agent2
    Config --> Agent3
    Config --> Agent4
    Config --> Redis
    Config --> Postgres
    Config --> Chroma
```

## Notes

- External traffic enters through `Kong`.
- Each agent runtime exposes A2A-style JSON-RPC, synchronous invoke, NDJSON streaming, SSE, WebSocket, and stdio bridge endpoints.
- Agents do not talk to NATS directly. They use the internal gRPC broker facade.
- `Redis` is the hot state and agent-memory layer.
- `PostgreSQL` is the relational and scheduled-work layer.
- `ChromaDB` is the shared retrieval store.

## Relations

- part_of [[Project Index]]
- documents [[Distributed A2A Helm Scaffold]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-05-28 | Frontmatter normalized (entity_type, status, created, modified; namespaced tags, `diagram`/`mermaid` → `artifact/diagram`); added Relations block. | Note lacked required frontmatter and graph links. | KB curation sweep — projects/ partition. |
