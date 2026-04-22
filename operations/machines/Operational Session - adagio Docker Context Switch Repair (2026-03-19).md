---
title: Operational Session - adagio Docker Context Switch Repair (2026-03-19)
type: note
permalink: kb/operations/machines/operational-session-adagio-docker-context-switch-repair-2026-03-19
tags:
- project/agent-kb
- domain/docker
- domain/context
- domain/ssh
- domain/tls
- machine/adagio
- machine/largo
- status/draft
---

# Operational Session - adagio Docker Context Switch Repair (2026-03-19)

## Purpose and Scope
Capture the starting state, execution plan, branch/rollback policy, working folders, and early evidence for diagnosing and repairing the failing `adagio` Docker context switch from this Mac into the Windows Docker Desktop host.

## User-Reported Symptom
From the local shell on 2026-03-19:

```text
└─[~/orbstack]> d context ls
NAME         DESCRIPTION                               DOCKER ENDPOINT                                ERROR
adagio       Windows Docker Desktop                    ssh://adagio
default      Current DOCKER_HOST based configuration   unix:///var/run/docker.sock
orbstack *   OrbStack                                  unix:///Users/luke/.orbstack/run/docker.sock

Head "http://docker.example.com/_ping": net/http: HTTP/1.x transport connection broken: malformed HTTP response "".
* Are you trying to connect to a TLS-enabled daemon without TLS?

Get "http://docker.example.com/v1.51/containers/json": net/http: HTTP/1.x transport connection broken: malformed HTTP response "".
* Are you trying to connect to a TLS-enabled daemon without TLS?
```

## Starting State Before Any Repair Edits
### Local machine (`largo` / this Mac)
- Working repo path observed by shell: `/Users/luke/docker`
- User-facing repo path in task context: `/Volumes/MACDEV/docker`
- Current git branch: `development`
- Current git HEAD before repair work: `236858142dfe43d6a78ebe40c2ffa700fc953dce`
- Local git worktree state before repair work: clean
- Local Docker contexts currently present:
  - `adagio` -> `ssh://adagio`
  - `default` -> `unix:///var/run/docker.sock`
  - `orbstack` -> `unix:///Users/luke/.orbstack/run/docker.sock`
- Local `~/.docker/config.json` currently records `"currentContext": "adagio"`
- Local `~/.docker/dswitch-local-context` currently contains `default`
- Direct `docker context inspect adagio` shows a clean SSH-backed context object with no `docker.example.com` host embedded in context metadata
- Inference at this stage: the `docker.example.com` failure is more likely coming from wrapper logic or exported Docker client environment than from the saved `adagio` context definition itself

### Remote machine (`adagio`)
- Canonical project root from existing KB notes: `C:\Users\me\dev\docker`
- Remote host profile from KB indicates:
  - Windows 11 Home host `ADAGIO`
  - Docker Desktop version `4.52.0`
  - Docker Engine version `29.0.1`
  - Active engine context observed on 2026-03-17 audit: `desktop-linux`
  - LAN IP `10.4.10.15`
- Initial SSH probe on 2026-03-19 reached `adagio\me` successfully, but the host defaulted into a PowerShell profile surface rather than a POSIX shell, so remote repo path verification and git-state capture still need a PowerShell-native command pass

## Relevant Existing Knowledge Loaded Before Repairs
- [[Machine Profile - adagio]]
- [[Local Network Docker Registry on adagio]]
- [[Docker Registry Helper Commands on adagio and tyggr]]
- [[Operational Session - largo SSH to adagio and tyggr in WSL Mirrored Mode (2026-03-15)]]
- [[Operational Session - Local Docker Registry Bootstrap and Registry Auth Repair on adagio and tyggr (2026-03-16)]]
- [[Operational Audit - adagio LAN Registry Live State and Mirror Gap (2026-03-17)]]
- Local repo manual: `docs/local-private-registry-manual.md`

## Working Folders and Branch Policy
### Local machine plan
- Repo folder: `/Users/luke/docker`
- New repair branch to create from current base: `fix/adagio-context-switch-20260319`

### Remote machine plan
- Expected repo folder: `C:\Users\me\dev\docker`
- Matching repair branch to create from the remote repo's current base after verification: `fix/adagio-context-switch-20260319`

### Branch/reset rule for iterate-until-success
- Every attempt starts from the original base commit captured before that attempt.
- If an attempt fails badly enough to restart, create a brand-new branch from the original base and leave the abandoned branch untouched.
- Do not recycle a failed branch for the next attempt.
- Commit early and often so each verified sub-step has a rollback point.

## Execution Plan
1. Use a persistent remote worker on `adagio` over SSH and keep local ownership for Mac-side diagnosis and integration.
2. Verify remote repo path, remote git base, remote docker state, and the exact command/wrapper path that implements the failing context switch.
3. Research Docker context/env precedence with current upstream docs and official references.
4. Patch the smallest correct layer on each machine, preferring wrapper/env cleanup over rewriting healthy context metadata.
5. Commit often on both new branches.
6. Verify end-to-end that switching to `adagio` yields a working Docker client session rather than `docker.example.com` TLS-placeholder traffic.
7. Update this note with final root cause, commands, branch SHAs, and validation evidence.

## Early Hypotheses To Test
- A helper wrapper around `docker` or `docker context use` is exporting `DOCKER_HOST=http://docker.example.com` or related TLS env vars.
- The `d` command may be layering its own context abstraction over Docker CLI context state and is out of sync with `~/.docker/config.json`.
- A shell profile or cross-machine bootstrap script is leaving stale Docker client env active when switching between `default`, `orbstack`, and `adagio`.
- The registry/TLS work on `adagio:5443` is adjacent context, but the present symptom points first to Docker daemon connection selection rather than registry auth itself.

## Evidence Registry
| Date | Context | Concrete Example | Source |
|------|---------|------------------|--------|
| 2026-03-19 | Local repo/git baseline | branch `development`, HEAD `236858142dfe43d6a78ebe40c2ffa700fc953dce`, clean worktree | local `git rev-parse` + `git status` |
| 2026-03-19 | Local Docker context baseline | `adagio` context stored as `ssh://adagio`; `currentContext` persisted as `adagio`; `dswitch-local-context` file contains `default` | local `docker context inspect`, `docker context ls`, `cat ~/.docker/config.json`, `cat ~/.docker/dswitch-local-context` |
| 2026-03-19 | User-visible failure signature | requests to `http://docker.example.com/_ping` and `/v1.51/containers/json` fail with malformed HTTP response and TLS warning text | user-provided terminal output |
| 2026-03-19 | Remote reachability preflight | SSH reached `adagio\me`, but repo-path probing failed because the remote shell surface was PowerShell-oriented rather than POSIX-oriented | local SSH preflight |

## Open Questions
- What exact implementation provides the `d` command on the local machine?
- Which variable or wrapper resolves to `docker.example.com` during the failing switch?
- Is the mismatch local-only, remote-only, or a coordinated wrapper on both machines?
- Does the remote machine carry matching helper logic that must be patched in parallel?

## Relations
- related_to [[Machine Profile - adagio]]
- related_to [[Local Network Docker Registry on adagio]]
- related_to [[Docker Registry Helper Commands on adagio and tyggr]]
- related_to [[Operational Session - largo SSH to adagio and tyggr in WSL Mirrored Mode (2026-03-15)]]
- related_to [[Operational Session - Local Docker Registry Bootstrap and Registry Auth Repair on adagio and tyggr (2026-03-16)]]
- related_to [[Operational Audit - adagio LAN Registry Live State and Mirror Gap (2026-03-17)]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-19 | Created starting-state session note with branch policy, working folders, current evidence, and execution plan before repair edits. | User explicitly required plan, working folders, and all initial state to be documented in Basic Memory before starting the repair. | User request to diagnose and patch the failing `adagio` context switch with a persistent subagent and strict branch hygiene. |

## Verified Findings
- Root cause of the observed `docker.example.com` / malformed HTTP startup failure was local shell startup side effects in `/Users/luke/.bash/.dockerrc`, not remote `adagio` Docker Desktop state.
- Three alias definitions executed Docker command substitutions while the shell was being sourced:
  - `dcka`
  - `chefdown`
  - `dconkill_exited`
- That startup-time Docker traffic was enough to trigger the bogus `http://docker.example.com/...` requests seen around `d context ls` after switching to the `adagio` context.
- `~/.local/bin/dswitch` was reviewed and is relevant, but it was not the immediate cause of the malformed HTTP failure. It already protects direct Docker calls with `env -u DOCKER_HOST -u DOCKER_CONTEXT`.

## Local Fix
- Local git-backed dotfiles repo used for patch: `/Users/luke/.bash`
- Local branch: `fix/adagio-context-switch-20260319`
- Commit created: `b1017a9332f08b07143d80554358850a9f8155d1`
- Commit message: `Fix docker startup alias side effects`
- Patch converted the three problematic aliases into functions so Docker command substitutions execute only when the helper is invoked, not during shell startup.

## Remote Verification
- Persistent agent `Parfit` re-verified on `adagio`:
  - repo root: `C:\Users\me\dev\docker`
  - branch: `fix/adagio-context-switch-20260319`
  - HEAD: `236858142dfe43d6a78ebe40c2ffa700fc953dce`
- Remote `docker` resolves to `C:\Program Files\Docker\Docker\resources\bin\docker.exe`.
- Remote shell state did not show `DOCKER_HOST`, `DOCKER_CONTEXT`, `DOCKER_TLS_VERIFY`, or `DOCKER_CERT_PATH`; only `DOCKERHUB_PAT` was present.
- No remote-side patch was required based on verified state.

## Validation Notes
- After the local `.dockerrc` patch, fresh shell startup no longer reproduced the earlier `docker.example.com` malformed HTTP failure during initialization.
- `dswitch adagio` still succeeds.
- Docker's handling of the special `default` context remains awkward in non-interactive probes, so `config.json` alone is not a reliable signal for that context; official Docker docs confirm `default` is a special in-memory context and `DOCKER_HOST` / `DOCKER_CONTEXT` override the selected context.

## Remaining Caveats
- Shell startup is still noisy and appears to have unrelated slowness/hanging around other initialization paths (`gpa` missing, Conda initialization), but that is separate from the adagio Docker context-switch failure addressed here.
- `~/.local/bin` already had unrelated dirty state before this session, including `dswitch`, so no change was made there during this repair.

## Restart Iteration 2 (2026-03-19)
- User reported the context is still not operational because `dic` continues to fail in the `adagio` context even though `docker context ls` is clean.
- Per restart policy, local work is being restarted from the original base commit in a new branch in the relevant git-backed folder.
- Local working folder for this iteration: `/Users/luke/.bash`
- Original local base commit for restart: `585686abf4cc7cb0e9084dd65361194cc6c4b433`
- Planned local restart branch: `fix/adagio-context-switch-r2-20260319`
- Persistent remote partner reopened for this iteration: `Copernicus` (`019d08ca-8c92-7550-b09b-65e9368f5186`)
- Success criterion tightened by user: do not stop until `docker images` shows actual state in both `orbstack` and `adagio` contexts.

## Restart Iteration 2 Outcome
- Root cause of the remaining `adagio` failure was remote SSH transport corruption, not local Docker context metadata.
- From `largo`, `docker -D --context adagio images` showed the Docker client launching `ssh adagio docker system dial-stdio`.
- Directly running `ssh adagio docker system dial-stdio` reproduced the issue: the remote PowerShell bootstrap printed banner/env text to stdout before the Docker stdio stream, corrupting the transport and causing the earlier malformed HTTP / `docker.example.com` symptom on the local client.

## Remote Fix Applied
- Remote git-backed folder used for the actual fix: `G:\My Drive\Powershell_Master`
- Remote restart branch: `fix/adagio-shell-docker-context-20260319-r1`
- Remote commit: `640f6cfa31f1d571e34c7fcc1bf52f2c6747c830`
- Commit message: `Silence SSH command shell bootstrap`
- Remote file changed by Copernicus: `G:\My Drive\Powershell_Master\Microsoft.PowerShell_profile.ps1`
- Live profile copy synchronized so SSH uses the silent path: `C:\Users\me\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1`
- Remote bootstrap source responsible for the noisy output was identified in `G:\My Drive\Powershell_Master\bootstrap_scripts\005-ENV_variables.ps1`.

## Verification After Remote Fix
- `docker --context adagio images --format '{{.Repository}}:{{.Tag}}' | head -n 10` now returns actual image state from `largo`.
- Plain `docker images --format '{{.Repository}}:{{.Tag}}' | head -n 10` also works again when the current context is `adagio`.
- Example returned rows on `adagio` include:
  - `adagio:5443/wxt-ollama:cpu`
  - `wxt-ollama:cpu`
  - `adagio:5443/dpage/pgadmin4:latest`
  - `dpage/pgadmin4:latest`
  - `adagio:5443/registry:3`

## OrbStack Recovery
- During final verification, the `orbstack` context failed separately because the local OrbStack daemon/socket was unhealthy and then could not restart due to disk pressure (`panic: rename ... vmgr.log ... no space left on device`).
- Minimal local cleanup performed: removed `~/Library/Caches/com.todesktop.230313mzl4w4u92.ShipIt` to free space.
- After cleanup and explicit restart, `orbctl status` returned `Running` and `docker --context orbstack images --format '{{.Repository}}:{{.Tag}}' | head -n 10` returned actual local image state.
- Example returned rows on `orbstack` include:
  - `timescale/timescaledb-ha:pg18-all`
  - `dmcp-mcp-filesystem:latest`
  - `dmcp-mcp-codex:latest`
  - `pgvector/pgvector:0.8.2-pg18`
  - `qdrant/qdrant:latest`

## Final State
- Success criterion satisfied: `docker images` shows actual state in both `adagio` and `orbstack` contexts from `largo`.
- Persistent remote partner `Copernicus` remains open pending user sign-off.
