---
title: SSH Remote Access Recovery and Operations Manual (Windows/Linux/macOS)
created: 2026-02-18
modified: 2026-03-15
type: instruction
entity_type: instruction
status: evergreen
permalink: instructions/ssh-remote-access-recovery-and-operations-manual-windows-linux-mac-os
tags:
- project/agent-kb
- domain/networking
- domain/ssh
- domain/windows-openssh
- status/evergreen
- op/update-node
- source/largo
- machine/largo
---

# SSH Remote Access Recovery and Operations Manual (Windows/Linux/macOS)

## Agent Curation Metaprompt (Adapted)
ROLE: You are the operations curator for SSH connectivity and remote shell reliability across mixed OS fleets. This manual is a living operational artifact.

GOVERNING PRINCIPLES:
1. ACCUMULATE, NEVER SUMMARIZE AWAY. Preserve concrete failure signatures, commands, and outputs.
2. STRUCTURED INCREMENTAL UPDATES ONLY. Update targeted sections; avoid full rewrites.
3. EVIDENCE-LINKED ENTRIES. Every runbook update must include date, context, example, and source.
4. SELF-REFERENTIAL VERIFICATION. Check this manual for existing patterns before adding new guidance.
5. EVOLUTION LOG IS MANDATORY. Append every meaningful change.
6. TRIGGER CONDITIONS FOR UPDATE. Update this manual when: (a) a new SSH failure mode appears, (b) a host OS/OpenSSH version changes, (c) security baseline changes, (d) a recovery run reveals missing steps, (e) fleet topology changes.
7. WHAT NOT TO CHANGE. Do not remove Anti-Pattern Registry entries; deprecate them with reason.
8. QUALITY SIGNAL TRACKING. Record MTTR, first-pass recovery rate, and recurring root causes.

## Purpose and Scope
This manual is the operational standard for diagnosing and restoring SSH access for all machines in a mixed network, with primary emphasis on Windows OpenSSH servers and PowerShell access.

Goals:
- Restore SSH reachability, authentication, and usable interactive shells.
- Normalize server configs to predictable, testable baselines.
- Provide repeatable commands and acceptance checks.
- Reduce recurrence through observability and change control.

Out of scope:
- Replacing SSH with RDP/WinRM-only workflows.
- Disabling host security controls as a permanent fix.

## Current Incident Snapshot (2026-02-18)
Context: Host alias `adagio` (`10.4.10.15`) from macOS client.

Observed evidence:
- SSH TCP reachability succeeded intermittently (`nc ... 22` successful in tests).
- Public key authentication succeeded for user `me`.
- `ssh -s adagio powershell` failed with `subsystem request failed on channel 0`.
- Interactive PTY sessions rendered `conhost.exe` control sequences but lacked reliable command interaction in this client execution context.
- `sshuser` account key auth failed while `me` succeeded (identity mapping issue is account-specific, not global network).

Inference from evidence:
- Primary failure is server-side session/shell/subsystem configuration, not basic network path or client key material.

## Canonical End State (Definition of Healthy)
Every managed host must pass all checks:
1. Port 22 reachable from authorized management clients.
2. `sshd` running and set to automatic start.
3. Host key trust stable (no unexpected key churn).
4. Public-key auth working for intended users.
5. Interactive shell is usable (commands execute, prompt responsive).
6. Non-interactive command execution returns output and exit status.
7. File transfer subsystem (SFTP/SCP) works or is explicitly disabled by policy.
8. Windows hosts: default shell and PowerShell subsystem configured intentionally.

## Recovery Architecture and Decision Tree
### Layer 0: Scope and Blast Radius
- Build host matrix: hostname, IP, OS, OpenSSH version, role, owner, last-known-good date.
- Classify outage: single host vs segment vs fleet-wide.

### Layer 1: Network Path
From each management client:
- `ping <host>` (if allowed)
- `nc -vz <host> 22` (or `Test-NetConnection <host> -Port 22` on Windows)
- DNS checks: forward and reverse records
- Route checks: gateway/VLAN changes, ACL updates, VPN split-tunnel drift

If Layer 1 fails, fix network/firewall first. Do not debug SSH auth until transport is stable.

### Layer 2: SSH Service Health (Host Local)
Windows:
- `Get-Service sshd`
- `Get-NetTCPConnection -LocalPort 22 -State Listen`
- `Get-NetFirewallRule -Name *OpenSSH*`

Linux/macOS:
- `systemctl status sshd` (or `sshd -T`)
- `ss -lntp | grep :22`
- Firewall daemon rules (`nft`, `ufw`, `pf` as applicable)

### Layer 3: Auth and Authorization
- Validate user exists and is not denied by sshd policy.
- Validate authorized keys file path and ACL/permissions.
- Confirm client is offering the intended key (`ssh -vvv host`).

### Layer 4: Session/Shell/Subystem
- Test non-interactive command: `ssh host "echo ok"`
- Test PTY shell: `ssh -tt host`
- Test subsystem explicitly: `ssh -s host sftp`, `ssh -s host powershell`

If auth succeeds but session fails, focus on shell/subsystem and sshd_config.

## Windows OpenSSH Healing Procedure (Primary)
### Step W1: Service and Firewall Baseline
Run in elevated PowerShell on server:
```powershell
Get-Service sshd
Set-Service -Name sshd -StartupType Automatic
Start-Service sshd
Get-NetFirewallRule -Name *OpenSSH* | Format-Table Name,Enabled,Direction,Action
```

### Step W2: Validate sshd_config
Path: `C:\ProgramData\ssh\sshd_config`

Required checks:
- No malformed directives.
- Intended `Subsystem sftp` present unless policy disables file transfer.
- If PowerShell remoting subsystem is required, include:
```text
Subsystem powershell C:/progra~1/powershell/7/pwsh.exe -sshs
```

### Step W3: Default Shell Registry Baseline
```powershell
Get-ItemProperty HKLM:\SOFTWARE\OpenSSH |
  Select-Object DefaultShell,DefaultShellCommandOption,DefaultShellEscapeArguments
```
Set explicitly when needed:
```powershell
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -PropertyType String -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShellCommandOption -Value "-c" -PropertyType String -Force
```
Restart sshd after changes:
```powershell
Restart-Service sshd
```

### Step W4: Admin Key Path and ACL Model
If authenticating as administrator-class users, verify admin key path and ACL requirements (commonly `C:\ProgramData\ssh\administrators_authorized_keys`).

### Step W5: UAC Remote Restrictions and Elevation Reality
Important behavior: local admin accounts can receive filtered (non-elevated) remote tokens under UAC remote restrictions.

If operations require full admin token remotely, evaluate controlled use of:
- `LocalAccountTokenFilterPolicy` policy adjustment (security-reviewed).
- JEA endpoint or constrained administrative workflow.
- Scheduled-task elevation pattern for specific maintenance commands.

Do not assume SSH login equals elevated PowerShell.

### Step W6: Logging for Root Cause Isolation
Temporarily increase logging in `sshd_config`:
```text
SyslogFacility LOCAL0
LogLevel DEBUG3
```
Restart service, reproduce failure, collect logs, then reduce verbosity after resolution.

## Linux/macOS Host Healing Procedure
### Step U1: Daemon and Config Test
```bash
sudo sshd -t
sudo systemctl restart sshd
sudo systemctl status sshd
```

### Step U2: Permission Hygiene
- `~/.ssh` typically `700`
- `authorized_keys` typically `600`
- Correct file ownership

### Step U3: MAC/RBAC
- SELinux/AppArmor policy denials
- PAM/mfa modules causing post-auth session drops

### Step U4: Shell Validity
- User shell exists and is executable
- No broken profile startup scripts causing immediate exit

## Fleet-Wide Network Recovery Workflow
1. Build target inventory and parallel-check port 22 reachability from at least two management vantage points.
2. Identify shared-failure boundaries (same VLAN, same gateway, same firewall policy object).
3. Roll forward minimal network fixes first (ACL, route, NAT, DNS).
4. Re-test transport before touching host auth config.
5. Execute per-host SSH service/auth/shell remediation in batches.
6. Record pass/fail state in host matrix after each stage.

## Verification Matrix (Must Pass)
For each host alias:
1. `ssh -vvv <alias> exit`
2. `ssh -tt <alias> "whoami"`
3. `ssh <alias> "echo SHELL_OK"`
4. `ssh -s <alias> sftp` (if enabled)
5. Windows PowerShell path test (if required): subsystem or default-shell behavior validated
6. Negative test: unauthorized key/user is rejected

## Operational Guardrails
- Never edit multiple trust/auth layers at once without checkpointing.
- Keep host key changes explicit and documented.
- Keep one known-good break-glass account per platform.
- Use change windows for broad sshd_config edits.

## Anti-Pattern Registry
| Anti-pattern | Impact | Corrective Action |
|---|---|---|
| Assuming auth success means shell success | Misses subsystem/shell failures | Always run session and subsystem tests after auth tests |
| Mixing network and auth changes simultaneously | Unclear root cause | Follow layered recovery order |
| Treating “admin group member” as “elevated token” on Windows | Privilege failures persist | Validate UAC/token behavior explicitly |
| Enabling DEBUG logging permanently | Noise/perf/security concerns | Use temporary debug windows and revert |
| Per-host ad hoc fixes with no baseline | Drift and repeated outages | Enforce canonical baseline + matrix verification |

### Anti-Pattern Evidence Log
| Anti-pattern | Date | Context | Concrete Example | Source |
|---|---|---|---|---|
| Assuming auth success means shell success | 2026-02-18 | `adagio` post-auth failure triage from macOS client | `ssh -vvv adagio` authenticated as `me`, but `ssh -s adagio powershell` returned `subsystem request failed on channel 0` | Local Validation Trace (`adagio`, 2026-02-18); PowerShell SSH remoting docs |
| Mixing network and auth changes simultaneously | 2026-02-18 | Mixed-fleet outage handling procedure | Manual requires Layer 1 transport re-test (`nc -vz <host> 22`) before touching auth/shell settings to avoid ambiguity | Fleet-Wide Network Recovery Workflow; OpenSSH troubleshooting guidance |
| Treating “admin group member” as “elevated token” on Windows | 2026-02-18 | Windows privilege model in remote SSH sessions | Manual documents filtered remote tokens and requires explicit evaluation of `LocalAccountTokenFilterPolicy` for admin-required actions | Microsoft UAC remote restrictions documentation |
| Enabling DEBUG logging permanently | 2026-02-18 | SSH service diagnostics hardening | Manual prescribes temporary `LogLevel DEBUG3` only for repro windows, then reversion after root-cause isolation | Windows OpenSSH troubleshooting guidance; `sshd_config(5)` |
| Per-host ad hoc fixes with no baseline | 2026-02-18 | Recurring SSH drift prevention across hosts | Canonical end-state checks plus verification matrix (`ssh -vvv <alias> exit`, subsystem tests) required for each host to prevent drift | Canonical End State + Verification Matrix in this manual |

## Source-Linked Evidence
- 2026-02-18, Microsoft Learn OpenSSH server config: default shell and admin key-path behavior (Windows OpenSSH configuration model).
- 2026-02-18, Microsoft Learn OpenSSH troubleshooting: service/firewall/logging and session limits guidance.
- 2026-02-18, Win32-OpenSSH wiki: `DefaultShell`, `DefaultShellCommandOption`, and shell behavior details.
- 2026-02-18, PowerShell docs: SSH remoting and PowerShell subsystem (`pwsh -sshs`) requirements.
- 2026-02-18, OpenBSD ssh/sshd man docs: authoritative SSH/sshd option semantics and debug conventions.
- 2026-02-18, Microsoft UAC remote restrictions doc: remote token filtering and `LocalAccountTokenFilterPolicy` behavior.

## Incident-Specific Notes: adagio
- Alias `adagio-pwsh` added client-side with forced TTY and PowerShell remote command for controlled testing.
- `adagio` key auth succeeded for `me`; `sshuser` key auth failed.
- `powershell` subsystem request failed, indicating missing/misconfigured subsystem on server.
- Remediation priority: server-side sshd subsystem/default-shell normalization and log-guided retest.

## Tunnel-Mediated Long-Lived SSH Sessions (Cloudflare local-network pattern)
When SSH is carried through a tunnel layer for browser-rendered or other long-lived remote terminal sessions, add one more diagnostic split before changing auth or shell policy:

1. Confirm the tunnel transport in use.
2. Confirm the local origin (`localhost:22` in this host pattern) is healthy.
3. Confirm SSH protocol keepalives are configured on both client and server.
4. Confirm the tunnel process you are editing is actually the production one.

### Practical guidance from `largo` (2026-04-21)
- Root cause on this host was not SSH auth failure. The dominant failure mode was a root `cloudflared` QUIC path showing repeated `timeout: no recent network activity` errors for long-lived browser terminal traffic.
- Cloudflare's current docs explicitly call out that idle SSH sessions can disconnect more often on QUIC when network devices aggressively time out UDP traffic, and recommend testing `protocol: http2` plus SSH keepalives.
- On this macOS host, the real daemon config was `/opt/homebrew/etc/ssh/sshd_config`, not `/etc/ssh/sshd_config`. A repair aimed at the wrong config tree would have produced fake progress.
- On this host, the production tunnel was the root LaunchDaemon, not a user-scoped Homebrew LaunchAgent. Always verify the live process args before editing files.

### Operational takeaway
For tunnel-mediated long-lived SSH sessions, treat transport selection and protocol keepalive policy as first-class recovery levers alongside classic SSH service, auth, and shell checks.

### Canonical local reference
For the concrete macOS implementation, use:
- [[Cloudflare Browser Rendering on largo]]
- [[Runbook - Cloudflare Browser Rendering Terminal Verification and Recovery on largo]]
- [[Operational Session - Cloudflare Browser Rendering Terminal Stabilization on largo (2026-04-21)]]

## Quality Signals
- Track MTTR per incident.
- Track first-pass recovery success rate.
- Track recurrence by root-cause category: network, sshd service, auth, shell/subsystem, policy/elevation.

### Quality Signal Log
| Date | Incident | Most Useful Sections | Gap / Improvisation | Outcome Signal | Source |
|---|---|---|---|---|---|
| 2026-04-21 | Tunnel-mediated browser-rendered terminal drops on `largo` | `Recovery Architecture and Decision Tree`, `Linux/macOS Host Healing Procedure`, `Tunnel-Mediated Long-Lived SSH Sessions (Cloudflare local-network pattern)` | Manual previously covered classic SSH failure domains better than Cloudflare-tunneled long-lived browser terminal sessions | Runbook now captures that transport selection (`http2` vs `quic`) and SSH keepalives can be decisive even when auth is healthy | Cloudflare docs + local repair evidence from `largo` |
| 2026-02-18 | `adagio` (Windows OpenSSH post-auth subsystem failure) | `Windows OpenSSH Healing Procedure`, `Recovery Architecture and Decision Tree`, `Verification Matrix` | Missing explicit quality-log schema before this remediation pass | Auth path confirmed; subsystem failure isolated to server-side config; remediation queued | Local Validation Trace (`adagio`, 2026-02-18) |
| 2026-02-18 | `adagio` (Windows OpenSSH post-auth subsystem failure) | `Windows OpenSSH Healing Procedure`, `Recovery Architecture and Decision Tree`, `Verification Matrix` | Missing explicit quality-log schema before this remediation pass | Auth path confirmed; subsystem failure isolated to server-side config; remediation queued | Local Validation Trace (`adagio`, 2026-02-18) |

## Client Error Ledger (largo -> adagio, 2026-02-18)
| Seq | Probe | Client-side observation | Operational implication | Evidence |
|---|---|---|---|---|
| 1 | `ssh -vvv ... adagio exit` | Auth succeeded as `me`, then session stalled in channel state until interrupted | Do not treat auth success as session success; force subsystem/shell validation next | Local validation trace + debug output |
| 2 | `ssh -vv ... adagio "whoami; ..."` | Exec accepted but read path failed (`Broken pipe`) with unreliable payload return | Indicates post-auth shell/exec pipeline instability on server | Local validation trace + debug output |
| 3 | `ssh -vv -tt ... adagio "cmd /c echo OK_FROM_CMD"` | Session emitted `conhost.exe` control sequences; output usability inconsistent in capture | PTY/conhost behavior must be accounted for during diagnostics | Local validation trace + debug output |
| 4 | `ssh -s adagio powershell` | `subsystem request failed on channel 0` | PowerShell subsystem absent/misconfigured; prioritize sshd subsystem/default-shell remediation | Local validation trace |
| 5 | `ssh -s adagio sftp` / `sftp adagio` / `scp ...` | Subsystem accepted then channel read failure/hang; transfer clients stalled | File-transfer and subsystem channel path also impacted; not only interactive shell | Local validation trace + client probes |
| 6 | Alias probe to `adagio-pwsh` with inline command | `Cannot execute command-line and remote command.` | Expected alias behavior; operators must not combine inline command with `RemoteCommand` alias | Local validation trace |
| 7 | Alias `ssh` (user `sshuser`) auth test | `Permission denied (publickey,password,keyboard-interactive)` while `me` succeeds | User/account-specific auth variance exists; remediate account path separately | Local validation trace |
| 8 | Bounded connectivity sequence | Mixed states observed: reachable, `Operation timed out`, stale session ended with `Host is down` | Transport/service intermittency coexists with shell/subsystem issues | Local validation trace + network probes |

## Operator Instruction for adagio-side Agent
Before applying on-host fixes, ingest this ledger and map each row to one of three buckets: `transport`, `authentication`, `session/subsystem`. Maintain bucketed remediation notes in the `adagio` machine profile stub and preserve command/output evidence.

## Evolution Log
| Date | Change | Reason | Trigger |
|---|---|---|---|
| 2026-04-21 | Added guidance for tunnel-mediated long-lived SSH sessions, including Cloudflare transport choice and the need to verify the actual production tunnel and `sshd` control planes | The manual needed to cover a now-proven failure mode where Cloudflare tunnel transport and wrong-daemon targeting dominated the observed browser terminal instability | User request to create detailed KB procedures from the completed Cloudflare browser-rendering stabilization work |
| 2026-02-18 | Created comprehensive fleet SSH recovery manual with Windows-first remediation and mixed-OS procedures | Repeated post-auth session failures and subsystem mismatch on `adagio` required standardized operations playbook | User request for holistic, agentic troubleshooting manual |
| 2026-02-18 | Created comprehensive fleet SSH recovery manual with Windows-first remediation and mixed-OS procedures | Repeated post-auth session failures and subsystem mismatch on `adagio` required standardized operations playbook | User request for holistic, agentic troubleshooting manual |
| 2026-02-18 | Curator remediation pass: frontmatter normalization, relations verb normalization, anti-pattern evidence log insertion, and quality signal log insertion | Previous audit identified missing required metadata and missing evidence/quality registries for curation protocol compliance | User-requested remediation fix under strict agent-kb curation protocol |
| 2026-02-18 | Added detailed client error ledger for `largo` -> `adagio` and explicit operator instruction for adagio-side triage bucketing | Ensure full client-side error context is preserved for the on-host remediation agent and prevent context loss between agents | User request for full accounting of client errors/debug state |
| 2026-03-15 | Added mirrored-mode topology update covering stale `tyggr` host mapping, `en0` route verification, `10.4.10.11:22` refusal, and unstable `10.4.10.15` banner exchange on ports `22` and `2222` | Preserve the newer mixed Windows/WSL mirrored-mode failure shape and connect it to the existing SSH recovery guidance | User request to research and curate the current incident state |

## Incident-Specific Notes: adagio and tyggr mirrored-mode update (2026-03-15)
- `largo` still had a stale `/etc/hosts` entry mapping `tyggr.local` and `tyggr` to `10.4.10.11`, which no longer matches the intended mirrored-mode topology.
- `route -n get 10.4.10.15` and `route -n get 10.4.10.11` both resolved through `en0`, which ruled out the active VPN route as the primary cause.
- `10.4.10.11` still answered ICMP but `22/tcp` returned `Connection refused`, confirming it is not the current SSH target for the WSL distro.
- `10.4.10.15:22` and `10.4.10.15:2222` both accepted TCP during repeated probes from `largo`, but bounded SSH clients timed out during banner exchange.
- Microsoft WSL mirrored-mode guidance and public issue history indicate that LAN access to WSL services still depends on explicit Windows Firewall and Hyper-V Firewall allowance, and the observed `:2222` behavior is consistent with stale or incomplete Windows-side exposure.
- Client-side remediation completed on `largo`: a `tyggr-wsl` alias was added targeting `adagio.local:2222` with `HostKeyAlias tyggr`; remaining work is Windows-side repair plus stale `/etc/hosts` cleanup.

### Additional authoritative references consulted on 2026-03-15
- GitHub CLI manual: `gh auth token` is the correct command to output the active token used by `gh`.
- Microsoft Learn: WSL networking / mirrored mode.
- Microsoft Learn: Hyper-V firewall configuration for WSL.
- GitHub issue: `microsoft/WSL #10769` (mirrored mode still requires firewall allowance for LAN access).
- GitHub issue: `microsoft/WSL #10597` (SSH exposure confusion in mirrored mode).

## Relations
- related_to [[Index - Cloudflare Browser Rendering Operations]]
- related_to [[Cloudflare Browser Rendering on largo]]
- related_to [[Runbook - Cloudflare Browser Rendering Terminal Verification and Recovery on largo]]
- related_to [[Operational Session - Cloudflare Browser Rendering Terminal Stabilization on largo (2026-04-21)]]

- related_to [[Fundamental – Agent KB]]
- related_to [[Agentic Knowledge Base System Overview]]
- related_to [[Action Taxonomy – Index]]
- implements [[Document Curation Metaprompt]]

## References (Authoritative URLs)
- Microsoft Learn: OpenSSH Server configuration for Windows
  - https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh-server-configuration
- Microsoft Learn: OpenSSH connection troubleshooting (`MaxStartups`, `MaxSessions`, diagnostics)
  - https://learn.microsoft.com/en-us/troubleshoot/windows-server/system-management-components/troubleshoot-openssh-connection-issues-maxstartups-maxsessions
- Win32-OpenSSH wiki: `DefaultShell` registry settings
  - https://github.com/PowerShell/Win32-OpenSSH/wiki/DefaultShell
- Win32-OpenSSH wiki: TTY/PTY support behavior
  - https://github.com/PowerShell/Win32-OpenSSH/wiki/TTY-PTY-support-in-Windows-OpenSSH
- PowerShell docs: SSH remoting and subsystem configuration (`pwsh -sshs`)
  - https://learn.microsoft.com/en-us/powershell/scripting/security/remoting/ssh-remoting-in-powershell
- OpenBSD ssh(1) manual (client-side semantics and debugging flags)
  - https://man.openbsd.org/ssh
- OpenBSD sshd_config(5) manual (server-side directives/subsystems)
  - https://man.openbsd.org/sshd_config
- Microsoft Learn: UAC remote restrictions / LocalAccountTokenFilterPolicy
  - https://learn.microsoft.com/en-us/troubleshoot/windows-server/windows-security/user-account-control-and-remote-restriction

## Local Validation Trace (adagio, 2026-02-18)
- `ssh -G adagio` resolved to `10.4.10.15`, user `me`, key auth preferred.
- `ssh -vvv adagio` showed successful public-key authentication against `OpenSSH_for_Windows_9.5`.
- `ssh -s adagio powershell` returned: `subsystem request failed on channel 0`.
- `ssh -vv -tt adagio` established PTY session with `conhost.exe` control-sequence output.
- `ssh -vv -tt ssh` (user `sshuser`) failed auth while `me` succeeded, confirming account-scoped auth variance.
