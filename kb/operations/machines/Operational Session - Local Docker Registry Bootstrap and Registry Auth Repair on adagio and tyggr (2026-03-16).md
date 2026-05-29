---
created: 2026-03-16
modified: 2026-05-28
entity_type: note
status: active
title: Operational Session - Local Docker Registry Bootstrap and Registry Auth Repair
  on adagio and tyggr (2026-03-16)
type: note
permalink: kb/operations/machines/operational-session-local-docker-registry-bootstrap-and-registry-auth-repair-on-adagio-and-tyggr-2026-03-16
tags:
- project/agent-kb
- domain/docker
- domain/registry
- domain/auth
- domain/shell
- machine/adagio
- machine/tyggr
- status/archive
- source/largo
- machine/largo
---

# Operational Session - Local Docker Registry Bootstrap and Registry Auth Repair on adagio and tyggr (2026-03-16)

## Purpose and Scope
Audit note for the session that built the local private Docker registry on `adagio`, added PowerShell and bash helper libraries, copied the active local image set into the registry, repaired shell-side auth precedence, and persisted the resulting artifacts across the `docker`, `powershell_profile`, and `.bash` repositories.

## Primary Outcomes
- Built a private local Docker registry stack under `C:\Users\me\dev\docker\registry`.
- Added the operator manual at `C:\Users\me\dev\docker\docs\local-private-registry-manual.md`.
- Added a PowerShell registry helper library at `G:\My Drive\Powershell_Master\bootstrap_scripts\039-Registry.ps1`.
- Added registry-selection behavior (`dic`, `dlr`) in `G:\My Drive\Powershell_Master\bootstrap_scripts\033-Docker.ps1`.
- Ported the helper surface into `~/.bash/.dockerrc` for `tyggr`.
- Copied the current local image set into the new registry.
- Disabled the noisy AWS/ECR bash startup behavior that was causing credential errors unrelated to the registry workflow.
- Repaired GHCR auth precedence so `gh auth token` is first and environment variables are only fallback.

## Registry Build Details
- Registry endpoint verified: `adagio:5443`
- Registry endpoint by LAN IP: `10.4.10.15:5443`
- Auth check succeeded against `/v2/` after stack bring-up.
- The registry was left in a working state with persistent data, certs, and htpasswd credentials.

## Shell/Auth Findings
### Bash-side AWS noise
Initial bash startup emitted `InvalidClientTokenId` because `.dockerrc` still contained eager AWS/ECR evaluation and `.bashrc` still attempted AWS version reporting.

Resolution:
- commented the eager ECR lines in `.dockerrc`
- commented AWS version reporting in `.bashrc`
- disabled `.awsrc` wholesale

Result:
- bash startup no longer emitted `InvalidClientTokenId`
- unrelated startup issues remained, including a `.bashrc` quote mismatch and missing local files for `kube-ps1` and `.ollama`

### GHCR auth shadowing
The effective failure was not GHCR itself but token precedence.

Observed state:
- `gh auth status` showed a valid keyring-backed GitHub CLI account
- `GH_TOKEN` in the environment was invalid
- plain `gh auth token` was being shadowed by that invalid env token

Resolution:
- PowerShell `Get-GhAuthToken` now temporarily clears `GH_TOKEN` and `GITHUB_TOKEN` before calling `gh auth token`
- WSL/bash now does the same before calling `gh auth token` directly or through a Windows-side PowerShell bridge

Result:
- `dlr ghcr` succeeded in both PowerShell and WSL after the precedence repair

## Registry Switch Verification Captured During Session
### PowerShell
- `dlr local` -> active registry became `local / adagio:5443`
- `dlr docker` -> active registry became `docker / docker.io`
- `dlr ghcr` -> active registry became `ghcr / ghcr.io`
- `dic` printed the expected header after each switch

### WSL / bash
- `dlr local` -> active registry became `local / adagio:5443`
- `dlr docker` -> active registry became `docker / docker.io`
- `dlr ghcr` -> active registry became `ghcr / ghcr.io`
- `dic` printed the expected header after each switch

## Repository Outcomes
### docker repo
- Added local registry stack and docs
- Commit recorded during the session: `2368581`

### powershell_profile repo
- Added local registry helpers and later GHCR precedence hardening
- Commits recorded during the session included `6b1ed32`, `9aba0c8`, and `0895f07`

### .bash repo
- Added bash helper surface, disabled AWS startup paths, added GHCR precedence hardening, and corrected an accidental local-only `node_modules` staging incident before final push
- The accidental `node_modules` commit never reached `origin`
- Final corrected pushed commit from the later cleanup was `e78ef21`

## Important Residual Gaps
- At session close, the machine profile for `adagio` was still a stub; it has since been superseded by [[Machine Profile - adagio]].
- Bash startup still has unrelated non-registry issues that were not resolved in this session.
- The session captured command behavior and repo state, but future agents should still validate live credentials before assuming Docker Hub or GHCR auth is still valid.

## Relations
- related_to [[Local Network Docker Registry on adagio]]
- related_to [[Docker Registry Helper Commands on adagio and tyggr]]
- related_to [[Operational Audit - adagio LAN Registry Live State and Mirror Gap (2026-03-17)]]
- related_to [[Client Onboarding for adagio LAN Docker Registry]]
- related_to [[Machine Profile - adagio]]
- related_to [[Operational Session - largo SSH to adagio and tyggr in WSL Mirrored Mode (2026-03-15)]]
- related_to [[Index – Agent KB]]
## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-16 | Created full session audit note for the local registry buildout, shell auth repairs, and repo persistence work. | This session changed machine-local infra, shell behavior, auth precedence, and multiple repos, so it needed a durable operational record. | User request to persist all important aspects of the session in the knowledge base. |