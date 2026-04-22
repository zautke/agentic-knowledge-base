---
title: Client Onboarding for adagio LAN Docker Registry
type: instruction
permalink: kb/operations/machines/client-onboarding-for-adagio-lan-docker-registry
tags:
- project/agent-kb
- domain/docker
- domain/registry
- domain/tls
- domain/network
- machine/adagio
- status/evergreen
---

# Client Onboarding for adagio LAN Docker Registry

## Purpose and Scope
Operator-facing onboarding note for LAN clients that need to authenticate to and trust the Docker registry on `adagio:5443`. This note distinguishes direct private-registry use from Docker mirror behavior and records the minimum client steps needed for successful push, pull, and catalog access.

## Endpoint Identity
- Preferred registry endpoint: `adagio:5443`
- Alternate LAN IP endpoint: `10.4.10.15:5443`
- Registry API base: `https://adagio:5443/v2/`
- Auth mode: HTTP Basic auth
- TLS trust anchor: `Local Docker Registry Root CA`

## Direct Registry Use
Use direct image naming whenever you want to push to or pull from the private registry explicitly.

```bash
docker login adagio:5443

docker tag my-app:latest adagio:5443/my-app:latest
docker push adagio:5443/my-app:latest
docker pull adagio:5443/my-app:latest
```

Docker treats `adagio:5443` as a registry host because the image prefix contains a `:` port delimiter.

## TLS Trust Requirements
Clients must trust the local CA before standard Docker and `curl` operations against `https://adagio:5443` will succeed without certificate warnings.

### macOS
- Preferred Docker-specific trust path: `~/.docker/certs.d/adagio:5443/ca.crt`
- OS-trust alternative: import the root CA into Keychain Access and mark it trusted for SSL.
- Restart Docker Desktop after installing the CA.

### Linux and WSL
- Docker trust path: `/etc/docker/certs.d/adagio:5443/ca.crt`
- Restart Docker Engine or Docker Desktop after installing the CA.

### Windows and Docker Desktop
- Docker trust path: `%USERPROFILE%\.docker\certs.d\adagio:5443\ca.crt`
- Restart Docker Desktop after installing the CA.

## Mirror Mode vs Direct Registry Mode
- Direct registry mode means clients tag images as `adagio:5443/<repo>:<tag>` and interact with the private registry explicitly.
- Mirror mode means a Docker daemon is configured with `registry-mirrors` so it checks `https://adagio:5443` before Docker Hub for compatible pulls.
- Mirror mode does not replace direct tagging semantics for pushes to the private registry.
- As of `2026-03-17`, `adagio` now has server-side pull-through-cache support configured via `proxy.remoteurl`, but LAN clients still need separate daemon-side mirror configuration if they should prefer `adagio` automatically.

## Temporary Compatibility Fallback
- Prefer proper CA trust over `insecure-registries`.
- Only use insecure-registry configuration as a temporary compatibility fallback when CA deployment is blocked.

## Verification
```bash
curl -I https://adagio:5443/v2/
docker login adagio:5443
docker pull adagio:5443/registry:3
```

Expected behavior:
- `curl` should reach the endpoint without TLS trust errors.
- unauthenticated registry requests may still return `401 Unauthorized`, which is normal when auth is enabled.
- `docker login` should succeed with the local registry credentials.

## Relations
- related_to [[Local Network Docker Registry on adagio]]
- related_to [[Operational Audit - adagio LAN Registry Live State and Mirror Gap (2026-03-17)]]
- related_to [[Docker Registry Helper Commands on adagio and tyggr]]
- related_to [[Machine Profile - adagio]]
- related_to [[Index – Agent KB]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-17 | Created client onboarding note for CA trust, login, and direct-vs-mirror registry usage. | The KB needed an operator-facing note for LAN clients after the live audit identified the CA trust gap and the registry role distinction. | User request to search the KB, update the note cluster, and carry out the registry-hub follow-up work. |