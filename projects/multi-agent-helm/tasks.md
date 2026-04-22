---
title: tasks
type: note
permalink: kb/projects/multi-agent-helm/tasks
tags:
- multi-agent
- helm
- tasks
---

# Tasks

## Completed

- Research stack and architecture
- Create greenfield scaffold
- Implement shared contracts and protobufs
- Implement gRPC broker
- Implement FastAPI runtime surface
- Add Helm charts and Dockerfiles
- Add Taskfile, Helmfile, k3d config, and scripts
- Add architecture documentation
- Add smoke tests
- Validate locally with `pytest` and `compileall`
- Prime basic memory and create project records

## Next

- Install `helm`
- Install `task` if missing
- Run `task up`
- Fix first end-to-end deployment issues
- Verify Kong, PostgreSQL, NATS, Redis, and Chroma behavior in-cluster

## Blocked

- Full cluster validation is blocked by missing `helm` in the current environment.
