---
title: Operational Session - largo SSH to adagio and tyggr in WSL Mirrored Mode (2026-03-15)
created: 2026-03-15
modified: 2026-03-15
type: reference
permalink: kb/operations/machines/operational-session-largo-ssh-to-adagio-and-tyggr-in-wsl-mirrored-mode-2026-03-15
entity_type: reference
status: draft
tags:
- project/agent-kb
- domain/networking
- domain/ssh
- domain/wsl
- domain/github
- domain/mcp
- machine/largo
- machine/adagio
- machine/tyggr
- status/draft
---

# Operational Session - largo SSH to adagio and tyggr in WSL Mirrored Mode (2026-03-15)

## Purpose and Scope
Capture the full cross-machine context for the 2026-03-15 investigation from `largo` into why SSH from the physical Mac into `adagio` and `tyggr` was failing after a WSL networking change from bridge mode to mirrored mode. This note also records the Codex/GitHub MCP authentication repair performed during the same session because stale GitHub credentials blocked repository-side research.

## Session Trigger
User-reported state:
- `adagio` (Windows host) and `tyggr` (WSL2 on `adagio`) could SSH into this Mac.
- `adagio` and `tyggr` could SSH back into each other.
- This physical Mac could not SSH into either target reliably.
- Prior known topology change: WSL networking moved from bridge mode to mirrored mode, so the distro should no longer be treated as having a distinct LAN IP from the Windows host.

## Starting State on `largo`
### Local SSH config
Observed from `~/.ssh/config` on 2026-03-15:
- `Host adagio` -> `HostName adagio.local`, `User me`
- `Host ssh` -> `HostName 10.4.10.15`, `User sshuser`
- No direct `Host tyggr` block remained in the current SSH config.
- A new remediation alias was added during this session:
  - `Host tyggr-wsl`
  - `HostName adagio.local`
  - `Port 2222`
  - `HostKeyAlias tyggr`
  - `IdentityFile ~/.ssh/id_ed25519`
  - `ConnectTimeout 5`

### Local hosts file
Observed from `/etc/hosts` on 2026-03-15:
- `10.4.10.11  tyggr.local tyggr`
- `10.4.10.15  adagio.local adagio.lan adagio`

Inference:
- The Mac still had a stale static name-to-IP mapping for `tyggr` pointing to the old bridge-mode address.

### Codex/GitHub MCP auth state
Observed from `~/.codex/config.toml` and `gh auth token`:
- The configured `GITHUB_PERSONAL_ACCESS_TOKEN` in Codex did not match the live GitHub CLI credential.
- `gh auth token` returned a non-empty 40-character token.
- `gh token auth` returned no token and is not the correct CLI form.
- The Codex config entry was updated during this session so the stored GitHub token matches `gh auth token`.

## Local Evidence Collected from `largo`
### Name resolution and routing
- `adagio.local` resolved to `10.4.10.15`.
- `tyggr.local` resolved to `10.4.10.11` because of the stale `/etc/hosts` entry.
- `route -n get 10.4.10.15` -> `interface: en0`
- `route -n get 10.4.10.11` -> `interface: en0`

Inference:
- The failure path was not caused by the active VPN default route; both relevant destinations were reached through the physical LAN interface (`en0`).

### Reachability and port probes
Observed on 2026-03-15 from `largo`:
- `ping 10.4.10.11` succeeded.
- `nc -vz 10.4.10.11 22` -> connection refused.
- `nc -vz 10.4.10.15 22` -> succeeded during repeated probes.
- `nc -vz 10.4.10.15 2222` -> succeeded during repeated probes.
- `ssh -o BatchMode=yes -o ConnectTimeout=5 me@10.4.10.15 exit` intermittently timed out during banner exchange.
- `ssh -o BatchMode=yes -o ConnectTimeout=5 -p 2222 luke@10.4.10.15 exit` also timed out during banner exchange.
- `ssh-keyscan -T 5 -p 2222 10.4.10.15` returned repeated host-down style failures during bounded retries.

Inference:
- `10.4.10.11` is still a live endpoint on the LAN, but it is not the current SSH target for `tyggr`.
- `10.4.10.15:22` is the Windows-side SSH endpoint and was intermittently reachable at the TCP layer but unstable at the SSH protocol layer.
- `10.4.10.15:2222` accepted TCP but did not reliably produce an SSH banner, which is consistent with a stale or broken forwarding path rather than a healthy WSL `sshd`.

### SSH auth specifics
Observed from `largo`:
- `sshuser@10.4.10.15` failed with `Permission denied (publickey,password,keyboard-interactive)`.
- `me@10.4.10.15` reached the server more successfully but remained unstable due to banner/session behavior.

Inference:
- There are two overlapping issues:
  1. Windows-host SSH service/session instability on port `22`.
  2. Broken or incomplete WSL mirrored-mode exposure for the distro path expected on port `2222`.

## External Research Used During the Session
### GitHub CLI
Official CLI documentation confirms:
- `gh auth token` is the correct command to output the active token for the current host/account.
- `gh auth token --hostname github.com` is appropriate when a specific host selection matters.
- `gh token auth` is not the documented command.

### WSL mirrored networking
Microsoft Learn guidance for WSL mirrored networking states:
- Mirrored mode is intended to allow direct LAN access to WSL services.
- Service exposure still depends on the service listening correctly and firewall policy allowing the traffic.
- Hyper-V firewall rules may need to be created explicitly for inbound traffic to WSL workloads.

### Relevant WSL issue history
The following public issue history matched the observed failure shape:
- `microsoft/WSL #10769`: mirrored mode still requires explicit firewall allowance for LAN access.
- `microsoft/WSL #10597`: users reported SSH confusion in mirrored mode when trying to expose a WSL `sshd` through the Windows host.

Inference from research:
- The current state is consistent with mirrored mode being enabled conceptually, while the actual WSL service exposure is incomplete or stale on the Windows side.

## Diagnosis
### Confirmed client-side problem
The Mac was still treating `tyggr` as `10.4.10.11` via `/etc/hosts`, which is stale after the move away from WSL bridge mode.

### Confirmed host-side problem
`adagio` itself is not healthy enough to serve as the stable gateway into Windows SSH or WSL SSH:
- Port `22` opens intermittently but times out during SSH banner exchange.
- Port `2222` opens at the TCP layer but does not behave like a healthy SSH listener.

### Most likely topology truth
- `adagio` should remain reachable at `10.4.10.15`.
- `tyggr` should be treated as a WSL service behind `adagio`, not as a separate LAN IP.
- The correct access model from `largo` is a dedicated host-side entry such as `adagio.local:2222`, with Windows Firewall and Hyper-V Firewall explicitly allowing that WSL listener.

## Actions Performed During This Session
### Codex control-plane repair
- Updated `~/.codex/config.toml` so `mcp_servers."github".env.GITHUB_PERSONAL_ACCESS_TOKEN` matches the live `gh auth token` credential.
- Rationale: GitHub MCP queries were failing with `Authentication Failed: Bad credentials`.

### Local SSH client repair
- Added `Host tyggr-wsl` to `~/.ssh/config` pointing to `adagio.local:2222` with `HostKeyAlias tyggr`.
- Added `repair-tyggr-mirrored-ssh.ps1` under `~/.ssh/` to be run from elevated PowerShell on `adagio`.
- The repair script removes stale `netsh interface portproxy` entries for `2222`, configures WSL `sshd` on `2222`, and adds the Windows Firewall and Hyper-V Firewall rules for that port.

### Constraint encountered
- `/etc/hosts` on the Mac is root-owned, and the stale `tyggr.local` entry could not be removed non-interactively during this session because sudo required a password.

## Required Manual Completion
1. On `largo`, remove the stale hosts entry:
```bash
sudo sed -i '' '/tyggr\.local tyggr/d' /etc/hosts
```
2. On `adagio`, run the repair script from elevated PowerShell:
```powershell
powershell -ExecutionPolicy Bypass -File C:\Users\me\.ssh\repair-tyggr-mirrored-ssh.ps1
```
3. After the Windows-side repair, validate from `largo`:
```bash
ssh adagio
ssh tyggr-wsl
```
4. Restart Codex or the GitHub MCP server process so the refreshed token in `~/.codex/config.toml` is actually picked up by a new session.

## Open Questions
- Why does `10.4.10.15:22` intermittently stall before completing SSH banner exchange on the Windows host?
- What process is currently accepting TCP on `10.4.10.15:2222` when the listener does not behave like a healthy SSH server?
- Is there a stale `portproxy`, alternate SSH daemon, or partially configured WSL forwarding path on `adagio`?

## Sources
- GitHub CLI manual: `gh auth token`
- Microsoft Learn: WSL networking / mirrored mode
- Microsoft Learn: Hyper-V firewall configuration for WSL
- GitHub issue: `microsoft/WSL #10769`
- GitHub issue: `microsoft/WSL #10597`

## Relations
- related_to [[SSH Remote Access Recovery and Operations Manual (Windows/Linux/macOS)]]
- related_to [[Machine Profile - largo]]
- related_to [[Machine Profile - adagio (Stub)]]
- related_to [[Index – Agent KB]]
- related_to [[Fundamental – Agent KB]]
- related_to [[Agentic Knowledge Base System Overview]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-15 | Created operational incident note capturing the full `largo` -> `adagio`/`tyggr` mirrored-mode SSH investigation plus the Codex GitHub MCP auth repair performed during the same session. | Preserve exact troubleshooting state, local evidence, external research, and pending manual steps without collapsing context into the older machine or runbook notes. | User request to research, solve, and write the full context into Basic Memory. |