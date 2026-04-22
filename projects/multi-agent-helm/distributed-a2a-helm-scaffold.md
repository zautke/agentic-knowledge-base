---
title: distributed-a2a-helm-scaffold
type: note
permalink: kb/projects/multi-agent-helm/distributed-a2a-helm-scaffold
tags:
- multi-agent
- helm
- kubernetes
- nats
- kong
- fastapi
- grpc
- redis
- postgres
- chromadb
---

# Distributed A2A Helm Scaffold

## Summary

Implemented a greenfield local-first Kubernetes scaffold in `distributed/` for a multi-agent platform with these core parts:

- `task up` entrypoint backed by PowerShell scripts
- `k3d` cluster config and `helmfile.yaml`
- Local Helm charts for `platform-config`, `nats`, `kong`, `postgres`, `redis`, `chromadb`, `broker`, and `agent-runtime`
- Python gRPC broker facade over NATS JetStream
- Shared FastAPI agent runtime with `/.well-known/agent-card.json`, `/a2a/jsonrpc`, `/invoke`, `/invoke/stream`, `/events/{task_id}`, `/ws`, and stdio session endpoints
- LangGraph-enabled supervisor runtime stub with optional Redis checkpointer wiring
- Generated gRPC stubs from `proto/broker.proto`
- Local smoke tests covering agent card, JSON-RPC task flow, NDJSON streaming, WebSocket roundtrip, and stdio bridge

## Base Image Decisions

- Alpine accepted: `redis:8-alpine`, `nats:2.11-alpine`
- Slim/vendor images used for: `python:3.12-slim-bookworm`, `postgres:18-bookworm`, `kong:3.13`, `chromadb/chroma`

## Notable Constraints

- Kong is implemented in DB-less OSS-compatible mode rather than enterprise AI plugins.
- PostgreSQL image includes pg_cron and PostGIS packages directly and scaffolds pgEdge repository integration with build args because package availability may vary by environment.
- Validation completed locally with `python -m pytest -q` and `python -m compileall packages/common/src services`.

## References

- PostgreSQL official image: https://hub.docker.com/_/postgres
- pgEdge release notes: https://docs.pgedge.com/enterprise/pep_rel_notes/
- FastAPI SSE docs: https://github.com/fastapi/fastapi/blob/master/docs/en/docs/tutorial/server-sent-events.md
- LangGraph Redis: https://github.com/redis-developer/langgraph-redis
- RedisVL: https://github.com/redis/redis-vl-python
- Chroma Docker docs: https://docs.trychroma.com/guides/deploy/docker
- Mem0 Redis docs: https://docs.mem0.ai/components/vectordbs/dbs/redis
- Kong install docs: https://developer.konghq.com/gateway/install/kubernetes/on-prem/
- A2A docs/spec: https://agent2agent.info/docs/ and https://github.com/a2aproject/A2A

## Documentation Records Added

- Project index: `kb/projects/multi-agent-helm/project-index`
- Initialization protocol execution: `kb/projects/multi-agent-helm/initialization-protocol-2026-04-11`
- Planning: `kb/projects/multi-agent-helm/planning`
- Session history: `kb/projects/multi-agent-helm/session-history`
- Task ledger: `kb/projects/multi-agent-helm/tasks`

## Repo Tracking Files

- `docs/PLANNING.md`
- `SESSION_HISTORY.md`
- `TASKS.md`
