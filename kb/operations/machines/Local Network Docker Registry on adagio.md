---
title: Local Network Docker Registry on adagio
type: reference
permalink: kb/operations/machines/local-network-docker-registry-on-adagio
tags:
- project/agent-kb
- domain/docker
- domain/registry
- domain/infrastructure
- machine/adagio
- machine/tyggr
- status/evergreen
---

# Local Network Docker Registry on adagio

## Purpose and Scope
Canonical reference for the self-hosted private Docker registry built on `adagio`, with client usage from Windows PowerShell and the `tyggr` WSL distro. This note captures the stack layout, endpoints, auth model, trust requirements, image catalog shape, and the repo artifacts that now define the local registry.

## Topology
- Host machine: `adagio`
- Secondary client environment: `tyggr` WSL
- Project root: `C:\Users\me\dev\docker`
- Stack root: `C:\Users\me\dev\docker\registry`
- Compose file: `registry/compose.yaml`
- Registry config: `registry/config/config.yml`
- Runtime env file: `registry/.env`
- Auth file: `registry/auth/htpasswd`
- Certificates: `registry/certs/`
- Data root: `registry/data/`
- Human manual: `docs/local-private-registry-manual.md`

## Published Endpoints
- Primary hostname endpoint: `adagio:5443`
- LAN endpoint: `10.4.10.15:5443`
- Registry API base: `https://adagio:5443/v2/`

## Security and Access Model
- Transport is TLS-protected with a locally generated CA and server certificate.
- Authentication uses `htpasswd` credentials.
- Docker clients must trust the generated CA to avoid TLS failures.
- The registry is intended for private LAN use rather than public Internet exposure.

## Live Audit Delta (2026-03-17)
- A direct SSH audit on `2026-03-17` confirmed that `adagio` is currently running Docker Desktop with active context `desktop-linux` and Docker `29.0.1`.
- The `lan-registry` container is still running on `0.0.0.0:5443` with TLS and `htpasswd` auth.
- The served certificate covers `adagio`, `localhost`, `10.4.10.15`, and `127.0.0.1`.
- At least one external LAN client still lacks trust for the local CA: a macOS probe to `https://adagio:5443/v2/` failed with `unable to get local issuer certificate`.
- Docker registry config on `adagio` still shows no client-side `registry-mirrors` entry for `adagio:5443`; only the built-in `hubproxy.docker.internal:5555` mirror-like entry was present in Docker's active registry config.
- During the follow-up remediation on `2026-03-17`, the registry server config was updated to include `proxy.remoteurl: https://registry-1.docker.io`, and the running `lan-registry` container was restarted with that configuration mounted.
- The registry now supports server-side pull-through-cache behavior, but LAN-wide mirror use still depends on client daemon configuration and CA trust rollout.
- Current-state audit trail: [[Operational Audit - adagio LAN Registry Live State and Mirror Gap (2026-03-17)]]
## Container and Runtime Identity
- Compose project name: `lan-registry`
- Container name: `lan-registry`
- Image: `registry:3`
- Verified operational state during the session: running and serving authenticated catalog responses.

## Canonical Supporting Artifacts
- `registry/compose.yaml` defines the local registry container and persistent mounts.
- `registry/config/config.yml` defines the registry runtime behavior.
- `docs/local-private-registry-manual.md` is the operator-facing how-to manual.
- `bootstrap_scripts/039-Registry.ps1` is the PowerShell command surface for initialization, lifecycle, trust, catalog inspection, and local image push.
- `bootstrap_scripts/033-Docker.ps1` holds the higher-level `dic` and `dlr` switching behavior on PowerShell.
- `~/.bash/.dockerrc` holds the WSL/bash port of the same registry helper surface.

## Known Catalog Shape Captured During Buildout
The registry was populated with the current local image set from the machine at build time, including:
- `braisenly/pg:latest`
- `braisenly/postgraphile:stable-dev`
- `dpage/pgadmin4:latest`
- `ghcr.io/yctimlin/mcp_excalidraw:latest`
- `httpd:2-alpine`
- `markitdown:latest`
- `mcp/desktop-commander:latest`
- `mcp/sequentialthinking:latest`
- `ollama/ollama:latest`
- `postgrest/postgrest:latest`
- `registry:3`
- `wxt-ollama:cpu`

## Operational Expectations
- Prefer `adagio:5443` for normal local use.
- Use the LAN-IP endpoint when name resolution is unavailable.
- Use the helper commands documented in [[Docker Registry Helper Commands on adagio and tyggr]].
- Use [[Client Onboarding for adagio LAN Docker Registry]] for CA trust rollout, `docker login`, and direct-vs-mirror onboarding on new client machines.
- Treat [[Operational Session - Local Docker Registry Bootstrap and Registry Auth Repair on adagio and tyggr (2026-03-16)]] as the buildout audit trail and [[Operational Audit - adagio LAN Registry Live State and Mirror Gap (2026-03-17)]] as the post-build verification and remediation audit trail.
## Relations
- related_to [[Machine Profile - adagio]]
- related_to [[Docker Registry Helper Commands on adagio and tyggr]]
- related_to [[Operational Session - Local Docker Registry Bootstrap and Registry Auth Repair on adagio and tyggr (2026-03-16)]]
- related_to [[Operational Audit - adagio LAN Registry Live State and Mirror Gap (2026-03-17)]]
- related_to [[Client Onboarding for adagio LAN Docker Registry]]
- related_to [[Index – Agent KB]]
## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-16 | Created canonical machine-local registry reference note for `adagio`/`tyggr`. | Future agents needed one durable place for endpoint, stack, security, and artifact truth. | User request to persist the full local private registry session into the KB. |
| 2026-03-17 | Added live-audit delta section and linked the post-build verification note. | The registry was operational, but the KB needed to distinguish a working private registry from an unconfigured mirror/hub state and capture the client trust gap. | User request to audit `adagio`, search the KB, and sync the findings back into the document base. |
| 2026-03-17 | Updated the canonical reference after enabling `proxy.remoteurl` and creating the client onboarding note. | The KB needed to reflect that server-side pull-through-cache support is now live while client mirror adoption and CA rollout remain open work. | User request to complete the three follow-up actions for the `adagio` registry cluster. |