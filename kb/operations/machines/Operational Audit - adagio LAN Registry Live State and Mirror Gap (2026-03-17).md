---
created: 2026-03-17
modified: 2026-05-28
entity_type: note
status: active
title: Operational Audit - adagio LAN Registry Live State and Mirror Gap (2026-03-17)
type: note
permalink: kb/operations/machines/operational-audit-adagio-lan-registry-live-state-and-mirror-gap-2026-03-17
tags:
- project/agent-kb
- domain/docker
- domain/registry
- domain/infrastructure
- domain/network
- domain/tls
- machine/adagio
- status/archive
- source/largo
- machine/largo
---

# Operational Audit - adagio LAN Registry Live State and Mirror Gap (2026-03-17)

## Purpose and Scope
Audit note for the live inspection of `adagio` after the initial local registry buildout. This session verified the current Docker Desktop engine state, the running registry container, the served TLS certificate, client trust behavior, Docker registry configuration, and the remaining gap between a functioning private registry and a true LAN-wide registry mirror / hub.

## Live Runtime Findings
- `adagio` is running Docker Desktop with active context `desktop-linux`.
- Remote Docker version observed during the audit: client `29.0.1`, server `29.0.1`, server OS `linux`.
- The registry container `lan-registry` is running from image `registry:3`.
- Published listener: `0.0.0.0:5443` and `[::]:5443`.
- Registry endpoint responds on `https://adagio:5443/v2/` with TLS and HTTP Basic auth.
- The registry is currently usable as a private authenticated registry.

## Certificate and Trust Findings
- The served certificate subject is `CN=adagio Registry`.
- The issuing CA is `CN=Local Docker Registry Root CA`.
- SAN coverage includes `DNS:adagio`, `DNS:localhost`, `IP:10.4.10.15`, and `IP:127.0.0.1`.
- Certificate validity observed during the audit: `2026-03-17` through `2031-03-17`.
- A macOS client probe to `https://adagio:5443/v2/` failed with `unable to get local issuer certificate`, confirming that at least one LAN client still lacks CA trust for the local registry.

## Docker Configuration Findings
- Docker Desktop settings at `C:\Users\me\AppData\Roaming\Docker\settings-store.json` do not show any registry mirror or registry-specific trust configuration.
- User Docker config at `C:\Users\me\.docker\config.json` contains auth material for `adagio:5443`, confirming host-local login state.
- User Docker daemon config at `C:\Users\me\.docker\daemon.json` only contains builder GC and `experimental: true`.
- `C:\ProgramData\Docker\config\daemon.json` was absent during the audit.
- `docker info` registry configuration still showed no configured `registry-mirrors` and no custom insecure registry entry for `adagio:5443`.
- Docker still reports the built-in `hubproxy.docker.internal:5555` entry, which means Docker Desktop's own proxy path remains the only recognized mirror-like registry path in the current engine state.
- After the follow-up remediation on `2026-03-17`, the registry server config on `adagio` was updated to include `proxy.remoteurl: https://registry-1.docker.io`, enabling server-side pull-through-cache behavior even though clients are not yet configured to prefer it automatically.
## Registry Container Findings
- Mounted paths:
  - data: `C:\Users\me\dev\docker\registry\data`
  - auth: `C:\Users\me\dev\docker\registry\auth`
  - certs: `C:\Users\me\dev\docker\registry\certs`
  - config: `C:\Users\me\dev\docker\registry\config\config.yml`
- `config.yml` enables filesystem storage, delete support, TLS, and `htpasswd` auth.
- As of the follow-up remediation on `2026-03-17`, `config.yml` now includes `proxy.remoteurl: https://registry-1.docker.io`.
- The running `lan-registry` container was restarted after the config change, and the mounted in-container config also showed the `proxy.remoteurl` section.
- The registry now supports pull-through cache behavior on the server side, but LAN-wide mirror behavior still depends on client daemon configuration and CA trust rollout.
## Interpretation
The earlier March 16 session successfully established a private LAN registry on `adagio`, and the March 17 follow-up now adds server-side pull-through-cache capability via `proxy.remoteurl`. The remaining gap is no longer the registry server itself; it is client adoption. Docker clients are not yet configured to prefer `https://adagio:5443` as a mirror, and non-host clients still require CA trust onboarding before they can use the registry cleanly.
## Required Remediation Direction
- Configure Docker clients or engine settings to use `https://adagio:5443` as a registry mirror where appropriate.
- Distribute `ca.crt` to LAN clients via OS trust store and/or Docker `certs.d/adagio:5443/ca.crt`.
- Preserve TLS mode rather than downgrading to insecure-registry mode unless there is a temporary compatibility requirement.
- Keep the registry on `adagio:5443/<repo>:<tag>` naming semantics for direct push/pull regardless of mirror adoption.
- Use [[Client Onboarding for adagio LAN Docker Registry]] as the operator-facing trust and login note for new clients.
## Knowledge-Base Update Strategy Started In This Session
### Entities to add or enrich
- `adagio` as an explicitly observed Docker Desktop Linux engine host rather than only a generic machine-local registry host.
- `lan-registry` as a concrete registry service instance with port binding, TLS cert identity, and mount topology.
- `adagio:5443` as a private registry endpoint that is distinct from a registry mirror role.
- `Local Docker Registry Root CA` as the trust anchor required by LAN clients.

### Relationship edges to add or strengthen
- `[[Local Network Docker Registry on adagio]]` related_to this audit note as the current-state delta.
- This audit note related_to `[[Operational Session - Local Docker Registry Bootstrap and Registry Auth Repair on adagio and tyggr (2026-03-16)]]` as the post-build verification successor.
- This audit note related_to `[[Docker Registry Helper Commands on adagio and tyggr]]` because helper semantics should stay aligned with whether the registry is direct-only or mirror-backed.
- The canonical registry reference should explicitly relate the endpoint role to Docker Desktop engine state and CA trust requirements.

### Observations to persist
- working private registry != configured pull-through cache / mirror
- host-local auth success != LAN-wide trust readiness
- Docker Desktop built-in hub proxy is still the only mirror-like entry observed in Docker registry config
- CA distribution is the current external-client blocker
- relation vocabulary drift (`related_to` vs `relates_to`) should not be increased while updating this cluster

## Relations
- related_to [[Local Network Docker Registry on adagio]]
- related_to [[Operational Session - Local Docker Registry Bootstrap and Registry Auth Repair on adagio and tyggr (2026-03-16)]]
- related_to [[Docker Registry Helper Commands on adagio and tyggr]]
- related_to [[Client Onboarding for adagio LAN Docker Registry]]
- related_to [[Machine Profile - adagio]]
- related_to [[Index – Agent KB]]
## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-17 | Created live-state audit note for the `adagio` LAN registry after direct SSH, runtime inspection, TLS verification, and external-doc comparison. | The KB already captured buildout state, but not the newer distinction between a working private registry and a true mirror/hub configuration. | User request to audit `adagio`, search the KB, and sync the findings into the document base. |
| 2026-03-17 | Updated the audit note after enabling `proxy.remoteurl` on the live registry and adding the client onboarding note. | The note needed to reflect that server-side pull-through-cache support is now active and that the remaining work is client trust plus mirror adoption. | User request to complete the three follow-up actions for the `adagio` registry cluster. |