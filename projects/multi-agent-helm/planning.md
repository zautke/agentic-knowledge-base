---
title: planning
type: note
permalink: kb/projects/multi-agent-helm/planning
tags:
- multi-agent
- helm
- planning
---

# Planning

## Status

- Initialization status: complete
- Scaffold status: implemented
- Local Python validation status: complete
- Full Kubernetes validation status: pending environment prerequisites

## Mission

Build a one-command local Kubernetes platform for a multi-agent, event-driven microservice ecosystem using `k3d`, `helmfile`, `NATS JetStream`, a Python gRPC broker facade, FastAPI agent runtimes, Kong in DB-less OSS-compatible mode, and PostgreSQL 18, Redis 8, and ChromaDB.

## Current Milestones

1. Project initialization: complete
2. Local control plane validation: pending `helm` and `task` availability
3. Runtime integration hardening: pending
4. Application-level integration: pending

## Immediate Next Steps

1. Install `helm` and `task` in the execution environment.
2. Run `task up`.
3. Fix any first-deploy chart, image, or rollout failures.
4. Add cluster-level smoke tests.
5. Begin replacing stub worker logic with real agent implementations.
