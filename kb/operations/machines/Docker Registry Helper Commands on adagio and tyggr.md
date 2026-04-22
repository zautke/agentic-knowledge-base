---
title: Docker Registry Helper Commands on adagio and tyggr
type: instruction
permalink: kb/operations/machines/docker-registry-helper-commands-on-adagio-and-tyggr
tags:
- project/agent-kb
- domain/docker
- domain/registry
- domain/shell
- domain/powershell
- machine/adagio
- machine/tyggr
- status/evergreen
---

# Docker Registry Helper Commands on adagio and tyggr

## Purpose and Scope
Canonical command-surface note for operating the local private Docker registry from Windows PowerShell and from the `tyggr` WSL/bash environment. This note records the helper functions, alias surface, registry-selection semantics, and the final auth precedence rules validated during the session.

## PowerShell Command Surface
Defined primarily in `G:\My Drive\Powershell_Master\bootstrap_scripts\039-Registry.ps1` and `033-Docker.ps1`.

### Registry lifecycle and inspection
- `Get-LanRegistryPaths` / `rgpa`
- `Get-LanRegistryEndpoint` / `rgep`
- `Initialize-LanRegistry` / `rgin`
- `Install-LanRegistryCa` / `rgca`
- `Start-LanRegistry` / `rgup`
- `Stop-LanRegistry` / `rgdn`
- `Get-LanRegistryStatus` / `rgst`
- `Login-LanRegistry` / `rgli`
- `Test-LanRegistry` / `rgts`
- `Get-LanRegistryCatalog` / `rgct`
- `Push-LocalImagesToLanRegistry` / `rgps`

### Registry selection and display
- `dic` prints a header in the form `Registry: <label> (<endpoint>)`, then a blank line, then the image/container inventory.
- `dlr` selects the active registry: `local`, `docker`, or `ghcr`.
- `dlr` with no argument reports the current registry selection.

## Bash / WSL Command Surface
Defined in `~/.bash/.dockerrc`.

### Local registry helpers
- `lan_registry_paths` / `rgpa`
- `lan_registry_endpoint` / `rgep`
- `lan_registry_init` / `rgin`
- `lan_registry_install_ca` / `rgca`
- `lan_registry_start` / `rgup`
- `lan_registry_stop` / `rgdn`
- `lan_registry_status` / `rgst`
- `lan_registry_login` / `rgli`
- `lan_registry_test` / `rgts`
- `lan_registry_catalog` / `rgct`
- `lan_registry_push_images` / `rgps`

### Registry selection and display
- `dic` prints the same current-registry header shape used by PowerShell.
- `dlr` selects `local`, `docker`, or `ghcr`.
- `dlr` with no argument prints the current selection.

## Auth Precedence Rules
### Local registry
- PowerShell: `Login-LanRegistry`
- Bash: `lan_registry_login`
- Behavior: login targets the local registry endpoint and uses the generated local credentials from the stack env/config.

### Docker Hub
- PowerShell uses token-variable login with `DOCKERHUB_PAT`, `DOCKERHUB_TOKEN`, then `bargs`.
- Bash uses `DOCKERHUB_PAT`, `DOCKERHUB_TOKEN`, then `bargs`, and on WSL also falls back to equivalent Windows environment variables through PowerShell lookup.

### GHCR
- `gh auth token` is first in the chain.
- Environment variables are backup only.
- The validated fallback set is `GH_TOKEN`, `GITHUB_PAT`, `GITHUB_DOCKER_TOKEN`, `GITHUB_PERSONAL_ACCESS_TOKEN`, and `GITHUB_TOKEN`.
- During this session an invalid `GH_TOKEN` environment override was found to shadow the working GitHub CLI keyring token. The final command surface now clears `GH_TOKEN` and `GITHUB_TOKEN` before calling `gh auth token`, then restores them after token retrieval.

## Bash-side AWS / ECR Disablement
The bash shell had AWS-related startup behavior that was unrelated to the local registry and was causing noisy auth failures.

### Disabled pieces
- eager `ecr_repo` evaluation in `~/.bash/.dockerrc`
- `ecrl` and `ecrpub` aliases in `~/.bash/.dockerrc`
- AWS version reporting in `~/.bash/.bashrc`
- the entire `~/.bash/.awsrc` file was disabled via a heredoc-style wrapper

### Result
- `InvalidClientTokenId` no longer appears during bash startup.
- Remaining startup issues are unrelated to AWS and were left as separate follow-up work.

## Verified Usage Pattern
```bash
# bash / WSL
source ~/.bash/.dockerrc

dlr local

dic

dlr docker

dic

dlr ghcr

dic
```

```powershell
# PowerShell
. 'G:\My Drive\Powershell_Master\bootstrap_scripts\033-Docker.ps1'
. 'G:\My Drive\Powershell_Master\bootstrap_scripts\039-Registry.ps1'

dlr local

dic

dlr docker

dic

dlr ghcr

dic
```

## Relations
- related_to [[Local Network Docker Registry on adagio]]
- related_to [[Operational Session - Local Docker Registry Bootstrap and Registry Auth Repair on adagio and tyggr (2026-03-16)]]
- related_to [[Operational Audit - adagio LAN Registry Live State and Mirror Gap (2026-03-17)]]
- related_to [[Client Onboarding for adagio LAN Docker Registry]]
- related_to [[Machine Profile - adagio]]
## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-16 | Created canonical helper-command and auth-precedence note for the local registry workflow. | Future agents needed durable command and auth truth across PowerShell and WSL. | User request to persist the registry helper surface and session outcomes in the KB. |