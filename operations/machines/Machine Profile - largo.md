---
title: Machine Profile - largo
type: reference
permalink: operations/machines/machine-profile-largo
created: 2026-02-18
modified: 2026-03-15
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/infrastructure
- domain/networking
- domain/ssh
- asset/machine
- machine/largo
---

# Machine Profile - largo

## Agent Curation Metaprompt (Adapted)
ROLE: You are the infrastructure curator for this machine profile. Preserve command-level evidence and update this note incrementally as host state changes.

GOVERNING PRINCIPLES:
1. ACCUMULATE, NEVER SUMMARIZE AWAY.
2. STRUCTURED INCREMENTAL UPDATES ONLY.
3. EVIDENCE-LINKED ENTRIES (date, context, example, source).
4. SELF-REFERENTIAL VERIFICATION (check existing entries before adding).
5. EVOLUTION LOG IS MANDATORY (append-only).
6. TRIGGER CONDITIONS FOR UPDATE: OS/network/SSH config changes, incident debug sessions, key/auth changes, alias changes.
7. WHAT NOT TO CHANGE: do not remove prior evidence; mark deprecated entries with rationale.
8. QUALITY SIGNAL TRACKING: log utility/gaps after each remediation use.

## Purpose and Scope
Canonical host/system and SSH client profile for machine `largo`, captured from direct local inspection during this session.

## 1) Machine Identity
| Field | Value | Evidence |
|---|---|---|
| Hostname | `largo.local` | `hostname` (2026-02-18) |
| Local username | `luke` | `whoami` / `id -un` |
| OS | `macOS 26.3 (Build 25D125)` | `sw_vers` |
| Kernel | `Darwin 25.3.0` | `uname -srvm` |
| Architecture | `arm64` | `uname -srvm` |
| Capture timestamp (UTC) | `2026-02-18T23:08:37Z` | `date -u +%Y-%m-%dT%H:%M:%SZ` |
| Capture timestamp (local) | `2026-02-18 16:09:30 -0700 MST` | `date +%Y-%m-%d %H:%M:%S %z %Z` |

## 2) Network Snapshot
### Primary IPs observed
- `en0`: `10.4.10.13` (active LAN interface)
- `utun7`: `10.5.0.2` (default route interface)
- `utun6`: `100.99.208.105`
- Virtual bridge addresses present: `bridge100 192.168.139.3`, `bridge101 192.168.215.0`, `bridge102 192.168.97.0`, `bridge103 192.168.147.0`, `bridge104 192.168.117.0`, `bridge105 192.168.107.0`

### Interface summary
- Active interfaces (status): `en0`, `awdl0`, `vmenet0..vmenet5`, `bridge100..bridge105`
- Inactive examples: `en1`, `en2`, `en3`, `en4`, `en5`, `en6`, `bridge0`, `anpi0`, `anpi1`, `ap1`

### Default route / DNS
- Default route destination: `default`
- Default route interface: `utun7`
- DNS nameservers observed in `scutil --dns` output: `100.100.100.100`, `100.64.0.2`, `1.1.1.1`, `1.0.0.1`

## 3) SSH Client Environment
### OpenSSH version
- `OpenSSH_10.2p1, OpenSSL 3.6.1 27 Jan 2026`

### Config files in use
- System client config: `/etc/ssh/ssh_config`
- System include directory: `/etc/ssh/ssh_config.d/*` (includes `100-macos.conf`)
- User client config: `~/.ssh/config`
- User config include: `Include ~/.orbstack/ssh/config`
- Known hosts file present: `~/.ssh/known_hosts`

### Key algorithms offered
- Supported key types from client capability query (`ssh -Q key`):
  - `ssh-ed25519`, `ssh-ed25519-cert-v01@openssh.com`
  - `sk-ssh-ed25519@openssh.com`, `sk-ssh-ed25519-cert-v01@openssh.com`
  - `ecdsa-sha2-nistp256`, `ecdsa-sha2-nistp256-cert-v01@openssh.com`
  - `ecdsa-sha2-nistp384`, `ecdsa-sha2-nistp384-cert-v01@openssh.com`
  - `ecdsa-sha2-nistp521`, `ecdsa-sha2-nistp521-cert-v01@openssh.com`
  - `sk-ecdsa-sha2-nistp256@openssh.com`, `sk-ecdsa-sha2-nistp256-cert-v01@openssh.com`
  - `ssh-rsa`, `ssh-rsa-cert-v01@openssh.com`
- Effective `pubkeyacceptedalgorithms` (from `ssh -G largo`): cert + `ssh-ed25519`/ECDSA/security-key + `rsa-sha2-512`/`rsa-sha2-256`.

### Agent status / socket
- `SSH_AUTH_SOCK=/Users/luke/.ssh/agent/s.RxSX5UJeLv.agent.W5YMSKmoOt`
- `SSH_AGENT_PID=16715`
- `ssh-add -l` returned 3 loaded keys (fingerprints observed; no private key content captured)

### Known host handling
- `StrictHostKeyChecking ask`
- `CheckHostIP no`
- `UserKnownHostsFile /Users/luke/.ssh/known_hosts /Users/luke/.ssh/known_hosts2`
- `GlobalKnownHostsFile /opt/homebrew/etc/ssh/ssh_known_hosts /opt/homebrew/etc/ssh/ssh_known_hosts2`

## 4) Local SSH Alias Map (`~/.ssh/config`)
| Alias | HostName | User | Notable options |
|---|---|---|---|
| `adagio` | `adagio.local` | `me` | `IdentityFile ~/.ssh/id_ed25519` |
| `adagio-pwsh` | `not present in current config` | `n/a` | `Legacy alias from 2026-02-18 evidence; no current host block` |
| `ssh` | `10.4.10.15` | `sshuser` | `IdentityFile ~/.ssh/id_ed25519` |
| `heim` | `heimdall.local` | `luke` | `IdentityFile ~/.ssh/id_ed25519` |
| `largo` | `largo.local` | `luke` | `IdentityFile ~/.ssh/id_ed25519` |
| `tyggr` | `not present in current config` | `n/a` | `Legacy direct alias removed; stale `/etc/hosts` still maps `tyggr.local` -> `10.4.10.11`` |
| `tyggr-wsl` | `adagio.local` | `luke` | `Port 2222`, `HostKeyAlias tyggr`, `ConnectTimeout 5`, `IdentityFile ~/.ssh/id_ed25519` |

## 5) Current Interoperability Notes (Observed This Session)
- `adagio` showed unstable reachability during non-interactive probes:
  - Initial probe emitted post-quantum warning banner from remote handshake.
  - Follow-up bounded probe returned `ssh: connect to host 10.4.10.15 port 22: Operation timed out`.
  - Stale session later terminated with `Connection to 10.4.10.15 port 22: Host is down`.
- `adagio-pwsh` alias is configured with `RemoteCommand`; probe with inline command returned `Cannot execute command-line and remote command.` (expected behavior when both are present).
- Interpretation: current symptoms are mixed transport/intermittent-host availability plus alias behavior constraints (`RemoteCommand`) for `adagio-pwsh`.

### 2026-03-15 Interoperability Delta
- Current `~/.ssh/config` no longer contains a direct `Host tyggr` block; instead a new `Host tyggr-wsl` alias was added to target `adagio.local:2222` with `HostKeyAlias tyggr` after mirrored-mode research.
- `/etc/hosts` on `largo` still maps `tyggr.local` and `tyggr` to `10.4.10.11`, which is now stale after the move away from WSL bridge mode.
- `route -n get 10.4.10.15` and `route -n get 10.4.10.11` both resolved to `interface: en0`, ruling out VPN path selection as the cause of the failure.
- `10.4.10.11:22` actively refused TCP while `10.4.10.15:22` and `10.4.10.15:2222` accepted TCP.
- Bounded SSH probes to `10.4.10.15:22` and `10.4.10.15:2222` timed out during banner exchange, indicating that the remaining blocker is on the Windows/WSL side rather than in the Mac routing path.

### 2026-04-21 Cloudflare Browser Rendering Delta
- The browser-rendered terminal path on `largo` now has an authoritative root `cloudflared` service path rather than an implicit user-scoped tunnel assumption.
- Active root tunnel config terminates `ssh.braisenly.com` to `ssh://localhost:22` and `opencode.braisenly.com` to `http://127.0.0.1:4096`.
- Stability baseline on this host now explicitly includes `ServerAliveInterval 30`, `ServerAliveCountMax 6`, `ClientAliveInterval 30`, `ClientAliveCountMax 6`, and `TCPKeepAlive no` on both sides.
- Later verification on 2026-05-10 superseded the earlier `sshd` path assumption: the live browser-rendered SSH path is the macOS socket-activated `com.openssh.sshd`, and authoritative config is `/etc/ssh/sshd_config` plus `/etc/ssh/sshd_config.d/*.conf`.
- Root `cloudflared` is pinned to `http2` with resolver pinning because pre-repair logs showed QUIC idle timeouts and resolver instability.
- Reusable local operational helpers now exist in `~/.ssh/apply-root-terminal-stability.sh` and `~/.ssh/diagnose-terminal-stack.sh`.
- Canonical domain notes for this new stack are [[Cloudflare Browser Rendering on largo]], [[Runbook - Cloudflare Browser Rendering Terminal Verification and Recovery on largo]], and [[Operational Session - Cloudflare Browser Rendering Terminal Stabilization on largo (2026-04-21)]].

### 2026-05-10 Cloudflare Access Session Delta
- SSH Access apps `ssh.braisenly.com` and `largo.braisenly.com/ssh` were updated from `24h` to `720h` (30 days) at the app level.
- App-local `Allow derp browser ssh` policies were also updated from `24h` to `720h`.
- The shared reusable Access policy remained at `24h` so unrelated protected surfaces were not broadened.

### 2026-04-29 Cloudflare Hostname Alias Delta
- Tunnel `3227fecc-8f59-41e4-9501-ab8235bc42f3` was updated to include additional ingress aliases:
  - `ssh.largo.braisenly.com` -> `ssh://localhost:22`
  - `opencode.largo.braisenly.com` -> `http://127.0.0.1:4096`
- DNS records were added for both aliases, proxied to `3227fecc-8f59-41e4-9501-ab8235bc42f3.cfargotunnel.com`.
- Tunnel config source moved from `local` to `cloudflare` for this tunnel after API configuration update.
- Immediate validation showed TLS handshake failures on both nested aliases, consistent with missing certificate coverage for `*.largo.braisenly.com`.
- At the same time, existing one-label hostnames remained mapped and healthy at the control-plane level:
  - `ssh.braisenly.com` -> `3227...cfargotunnel.com`
  - `opencode.braisenly.com` -> `9c40...cfargotunnel.com`

### 2026-04-30 Cloudflare Fallback Hostname Delta
- Access apps were added for one-label fallback hostnames:
  - `ssh-largo.braisenly.com`
  - `opencode-largo.braisenly.com`
- External edge checks returned `302` for those one-label hostnames (expected Access redirect path).
- Nested aliases (`ssh.largo.braisenly.com`, `opencode.largo.braisenly.com`) still failed TLS handshake.
- Advanced certificate API ordering remained blocked by credential scope; existing local tokens did not have SSL certificate write access.

### 2026-05-10 User `derp` Base-Hostname Recovery Delta
- User-local `opencode` installation was already present at `/Users/derp/.opencode/bin/opencode`; the missing piece was process supervision.
- A user LaunchAgent was added to keep the local web origin alive on `127.0.0.1:4096`:
  - `/Users/derp/Library/LaunchAgents/com.braisenly.opencode.web.plist`
  - `/Users/derp/.cloudflared/start_opencode_web.sh`
- The environment source of truth for this user was confirmed as `~/.bash/.env`, which is a symlink to `/Users/derp/vault/obsidian/comp/common_machines/global_env.md`.
- After loading the LaunchAgent, `opencode` returned `200` on `http://127.0.0.1:4096/` and the base public hostnames continued returning `302` through Cloudflare Access.

## 6) Evidence Registry
| Date (UTC) | Context | Concrete example | Source |
|---|---|---|---|
| 2026-05-10 | Base-hostname recovery for user `derp` on `largo` | User LaunchAgent `com.braisenly.opencode.web` restored `opencode` on `127.0.0.1:4096`; `ssh.braisenly.com` and `opencode.braisenly.com` both returned `302` through Access after recovery; SSH Access session for the `derp` browser path was extended from `24h` to `720h` | Local `launchctl`, local HTTP probe, public curl probes from `largo`, and Cloudflare Access API verification |
| 2026-04-30 | Fallback hostname activation under nested-subdomain TLS constraint | `ssh-largo.braisenly.com` and `opencode-largo.braisenly.com` returned `302` while nested aliases still failed TLS handshake; certificate API order attempts failed due auth/scope mismatch | Cloudflare Access app API, DNS/tunnel config API, and local curl probes from `largo` |
| 2026-04-29 | Hostname alias cutover for browser-rendered terminal stack on `largo` | Tunnel `3227...` ingress updated with `ssh.largo.braisenly.com` and `opencode.largo.braisenly.com`; DNS CNAMEs created; control plane healthy; direct HTTPS probes returned TLS handshake failures indicating nested-subdomain cert coverage gap | Cloudflare API (`/cfd_tunnel/.../configurations`, `/zones/.../dns_records`) plus local curl probes from `largo` |
| 2026-04-21 | Browser-rendered terminal stability capture on `largo` | Root `cloudflared` config showed `ssh.braisenly.com -> ssh://localhost:22`; logs showed pre-repair QUIC idle failures and post-repair `Initial protocol http2`; effective `sshd` config confirmed Homebrew path plus `ClientAliveInterval 30` / `ClientAliveCountMax 6` / `TCPKeepAlive no` | Local config inspection, `sshd -T`, `ssh -G`, and `/Library/Logs/cloudflare/cloudflared.err.log` on `largo` |
| 2026-02-18 | Host identity capture on `largo` | `hostname -> largo.local`, `whoami -> luke`, `sw_vers -> macOS 26.3` | Local shell commands on `largo` |
| 2026-02-18 | Host identity capture on `largo` | `hostname -> largo.local`, `whoami -> luke`, `sw_vers -> macOS 26.3` | Local shell commands on `largo` |
| 2026-02-18 | Network snapshot capture | `ifconfig` showed `en0 10.4.10.13`, default route on `utun7` | Local shell commands on `largo` |
| 2026-02-18 | SSH environment capture | `ssh -V -> OpenSSH_10.2p1`, `ssh -G largo` resolved known-host policy and key settings | Local shell commands on `largo` |
| 2026-02-18 | Alias inventory capture | `~/.ssh/config` host blocks for `adagio`, `adagio-pwsh`, `ssh`, `heim`, `largo`, `tyggr` | Local file inspection on `largo` |
| 2026-02-18 | Interop symptom capture | Timeout/host-down for `adagio`; command-line + `RemoteCommand` conflict on `adagio-pwsh` | Local SSH probes from `largo` |
| 2026-03-15 | Mirrored-mode topology drift capture | `/etc/hosts` still mapped `tyggr.local` -> `10.4.10.11`; `route -n get 10.4.10.15` and `route -n get 10.4.10.11` both used `en0`; `10.4.10.15:22` and `:2222` accepted TCP while bounded SSH clients timed out during banner exchange | Local shell + SSH probes from `largo` |

## 7) Validation Checklist
- [x] Machine identity captured with hostname/user/OS/kernel/arch/timestamp
- [x] Network snapshot captured with IPs, interfaces, default route, DNS
- [x] SSH client environment captured (OpenSSH version, config files, key algorithms, agent status, known-host handling)
- [x] Required local aliases documented (`adagio`, `adagio-pwsh`, `ssh`, `heim`, `largo`, `tyggr`)
- [x] Interoperability notes recorded from current session evidence

## 8) Handoff
This note is the authoritative current-state profile for `largo`. Re-run the same command set after OS/network/SSH config changes and append new evidence + evolution-log entries rather than replacing prior findings.

## 9) Client Error Ledger (`largo` -> `adagio`)
| Seq | Date (local) | Probe | Client-side observation | Interpretation | Source |
|---|---|---|---|---|---|
| 1 | 2026-02-18 | `ssh -vvv -o BatchMode=yes -o ConnectTimeout=8 -o ConnectionAttempts=1 adagio exit` | Transport + auth succeeded (`Authenticated ... using "publickey"`), then session did not complete cleanly and remained in channel exchange until manually interrupted | Post-auth session/shell path unstable on server side | Local SSH debug trace from `largo` |
| 2 | 2026-02-18 | `ssh -vv -o BatchMode=yes -o ConnectTimeout=8 adagio "whoami; ..."` | Exec accepted but channel read path failed (`read failed ... Broken pipe`) with missing expected command output | Server-side shell/exec handling inconsistency after auth | Local SSH debug trace from `largo` |
| 3 | 2026-02-18 | `ssh -vv -tt -o BatchMode=yes -o ConnectTimeout=8 adagio "cmd /c echo OK_FROM_CMD"` | Session closed with exit 0 while emitting `conhost.exe` control sequences; expected payload not reliably visible in client capture | PTY/conhost behavior is present and may mask or disrupt command-output expectations in this client context | Local SSH debug trace from `largo` |
| 4 | 2026-02-18 | `ssh -vv -o BatchMode=yes -o ConnectTimeout=8 -s adagio powershell` | `subsystem request failed on channel 0` | PowerShell SSH subsystem not configured/available on `adagio` | Local SSH debug trace from `largo` |
| 5 | 2026-02-18 | `ssh -vv -o BatchMode=yes -o ConnectTimeout=8 -s adagio sftp` | Subsystem accepted, then channel hit `read failed ... Broken pipe` and stalled behavior | File-transfer/session channel unstable post-subsystem start | Local SSH debug trace from `largo` |
| 6 | 2026-02-18 | `sftp -o BatchMode=yes -o ConnectTimeout=8 adagio` and `scp ... adagio:...` | Both clients stalled post-handshake warning with no usable transfer/session progression | Consistent with subsystem/session instability rather than key failure | Local client probes from `largo` |
| 7 | 2026-02-18 | `ssh -vv -tt ... ssh "cmd /c echo TEST_SSHUSER"` (alias `ssh`, user `sshuser`) | `Permission denied (publickey,password,keyboard-interactive)` while `me` on `adagio` succeeded | Account-scoped auth issue for `sshuser`, not global host-key mismatch | Local SSH debug trace from `largo` |
| 8 | 2026-02-18 | `ssh -tt -o BatchMode=yes -o ConnectTimeout=8 adagio-pwsh "cmd /c ..."` | Client error: `Cannot execute command-line and remote command.` | Expected alias behavior when `RemoteCommand` is set and inline command is also provided | Local client probe from `largo` |
| 9 | 2026-02-18 | Reachability sequence (`nc -vz 10.4.10.15 22`, bounded SSH probes) | Mixed states observed across attempts: reachable, timeout (`Operation timed out`), stale-session teardown (`Host is down`) | Intermittent transport/service availability in addition to post-auth session defects | Local network and SSH probes from `largo` |

## Quality Signals
- Track whether this profile reduced diagnosis time for cross-host SSH incidents.
- Track missing fields discovered during handoff to downstream agents.

### Quality Signal Log
| Date | Incident/Use | Most Useful Section(s) | Gap Found | Outcome Signal | Source |
|---|---|---|---|---|---|
| 2026-04-21 | Cloudflare browser-rendered terminal stabilization on `largo` | `3) SSH Client Environment`, `2026-04-21 Cloudflare Browser Rendering Delta`, `6) Evidence Registry` | Earlier machine profile did not explain that root `cloudflared` and Homebrew `sshd` were the authoritative production path for remote browser terminal work | Future agents can now land on the host note and pivot directly into the Cloudflare cluster instead of rediscovering the same control-plane split | Session curation after verifier pass on 2026-04-21 |
| 2026-02-18 | adagio triage handoff | `3) SSH Client Environment`, `9) Client Error Ledger` | Needed explicit curation and quality sections for strict protocol checks | Upstream context completeness improved for adagio-side agent preflight | Curator audit + this remediation pass |
| 2026-02-18 | adagio triage handoff | `3) SSH Client Environment`, `9) Client Error Ledger` | Needed explicit curation and quality sections for strict protocol checks | Upstream context completeness improved for adagio-side agent preflight | Curator audit + this remediation pass |

## Related Local Data Services
- [[Local Neo4j and Qdrant Container Operations on largo]]
- [[Operational Session - Codex Neo4j and Qdrant MCP Repair on largo (2026-03-15)]]

## Evolution Log
| Date | Change | Reason | Trigger |
|---|---|---|---|
| 2026-05-10 | Added user-`derp` LaunchAgent recovery delta and evidence for restoring the base Cloudflare browser-rendering hostnames without tunnel recreation. | Preserve the actual fix path: the tunnel plane was healthy and only the local `opencode` origin plus process supervision needed repair. | User request to bring the `ssh` and `opencode` tunnels back up for a new user on the same machine. |
| 2026-04-30 | Added one-label fallback-hostname delta, evidence row, and SSL-token scope blocker for nested-hostname cert remediation. | Preserve the operationally working fallback path and prevent repeated credential-scope debugging for advanced cert ordering. | Follow-up execution to restore largo endpoints after nested hostname TLS failures. |
| 2026-04-29 | Added hostname-alias delta and evidence for `ssh.largo.braisenly.com` / `opencode.largo.braisenly.com`, including cert-coverage failure mode. | Preserve operational context for the machine-scoped alias cutover and prevent repeated rediscovery of nested-subdomain TLS behavior. | User request to bring `ssh.largo.braisenly.com` and `opencode.largo.braisenly.com` online on this machine. |
| 2026-04-21 | Added browser-rendering transport delta, Cloudflare control-plane references, and related-note links for the stabilized iPad terminal path | The machine profile needed to reflect that `largo` now has a canonical browser-rendered terminal stack with root tunnel ownership and explicit SSH keepalive policy | User request to build the `cloudflare browser rendering` KB foundation and relate it back into existing infrastructure notes |
| 2026-02-18 | Created full machine profile for `largo` with host/system, network, SSH environment, alias map, and session interop notes | Establish canonical infra evidence record in `agent-kb` under strict curation protocol | User request for Curator-Infra machine profiling |
| 2026-02-18 | Created full machine profile for `largo` with host/system, network, SSH environment, alias map, and session interop notes | Establish canonical infra evidence record in `agent-kb` under strict curation protocol | User request for Curator-Infra machine profiling |
| 2026-02-18 | Added detailed client error ledger for `largo` -> `adagio` covering transport/auth/subsystem/session failures with command-level evidence | Ensure downstream `adagio` agent has full client-side failure context before on-host remediation | User request for complete client error accounting and protocol-compliant curation |
| 2026-02-18 | Added explicit agent curation metaprompt and quality-signal tracking sections | Resolve strict protocol gaps from curator audit without rewriting evidence sections | User directive to proceed with protocol remediation (`go`) |
| 2026-03-15 | Added mirrored-mode interoperability delta and evidence rows for stale `tyggr` hosts mapping, `en0` routing verification, and Windows/WSL banner-exchange failures | Preserve topology drift and current client-side evidence after the move from WSL bridge mode to mirrored mode | User request to research and curate the current SSH failure state |

## Relations
- related_to [[SSH Recovery Snapshot - largo and adagio (2026-04-28)]]
- related_to [[Index - Cloudflare Browser Rendering Operations]]
- related_to [[Cloudflare Browser Rendering on largo]]
- related_to [[Runbook - Cloudflare Browser Rendering Terminal Verification and Recovery on largo]]
- related_to [[Operational Session - Cloudflare Browser Rendering Terminal Stabilization on largo (2026-04-21)]]

- related_to [[SSH Remote Access Recovery and Operations Manual (Windows/Linux/macOS)]]
- related_to [[Index – Agent KB]]
- related_to [[Fundamental – Agent KB]]
