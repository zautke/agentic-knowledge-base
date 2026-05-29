---
title: Machine Profile - adagio
type: reference
permalink: kb/operations/machines/machine-profile-adagio
aliases:
- Machine Profile - adagio (Stub)
created: 2026-02-18
modified: 2026-03-16
status: evergreen
entity_type: reference
tags:
- project/agent-kb
- domain/infrastructure
- domain/networking
- domain/ssh
- domain/windows
- domain/wsl
- domain/docker
- asset/machine
- machine/adagio
- status/evergreen
---

# Machine Profile - adagio

## Purpose and Scope
Canonical host/system, SSH, WSL, and Docker profile for machine `adagio`, captured from direct local inspection on 2026-03-16 and promoted from the former stub note.

## 1) Machine Identity
| Field | Value | Evidence |
|---|---|---|
| Hostname | `ADAGIO` | `$env:COMPUTERNAME`, `systeminfo` |
| Local username | `adagio\me` | `whoami` |
| OS | `Microsoft Windows 11 Home` | `Get-ComputerInfo.OsName`, `systeminfo` |
| OS version / build | `10.0.26200 / Build 26200` | `Get-ComputerInfo`, `systeminfo` |
| Architecture | `64-bit`, `x64-based PC` | `Win32_OperatingSystem.OSArchitecture`, `systeminfo` |
| Manufacturer | `Dell Inc.` | `Win32_ComputerSystem.Manufacturer` |
| Model | `Inspiron 16 7610` | `Win32_ComputerSystem.Model`, `systeminfo` |
| CPU | `11th Gen Intel(R) Core(TM) i7-11800H @ 2.30GHz` | `Win32_Processor.Name` |
| BIOS version | `Dell Inc. 1.29.0` | `Win32_BIOS`, `systeminfo` |
| BIOS serial | `D2QGLG3` | `Win32_BIOS.SerialNumber` |
| Total physical memory | `31.74 GiB` (`32,507 MB` reported by `systeminfo`) | `Win32_ComputerSystem.TotalPhysicalMemory`, `systeminfo` |
| Capture timestamp (UTC offset surface) | `2026-03-16T23:05:17.7606156-06:00` | `Get-Date -Format o` |
| Capture timestamp (local) | `2026-03-16 23:05:17 -06:00` | `Get-Date -Format "yyyy-MM-dd HH:mm:ss zzz"` |
| Install date | `2025-11-16 22:43:50` | `Win32_OperatingSystem.InstallDate` |
| Last boot | `2026-03-12 13:08:56` | `Win32_OperatingSystem.LastBootUpTime` |

## 1A) Hardware and Firmware Notes
- `Get-ComputerInfo.WindowsProductName` returned `Windows 10 Home` even though `OsName` and `systeminfo` both reported `Windows 11 Home`.
- For this host, `OsName` plus `systeminfo` should be treated as the primary product-name surface and `WindowsProductName` as a legacy branding field.
- `systeminfo` reported `7,050 MB` available physical memory and `65,693 MB` virtual memory in use at capture time.

## 2) Network Snapshot
### Primary IPv4 identities observed
- `Ethernet`: `10.4.10.15/24` — primary LAN identity
- `vEthernet (WSL (Hyper-V firewall))`: `172.21.48.1/20` — Windows-side WSL virtual switch
- `NordLynx`: `10.5.0.2/16` — VPN tunnel
- `Tailscale`: `100.114.144.54/32` — overlay network
- `OpenVPN Data Channel Offload for NordVPN`: `10.100.0.2/22` — tunnel/offload path in tentative state during capture

### Default route and gateway
- Persistent default route: `0.0.0.0/0 -> 10.4.10.1`
- Active default path at capture: `10.4.10.1` via interface `10.4.10.15`

### DNS servers observed
- `Ethernet`: `1.1.1.1`, `1.0.0.1`
- `OpenVPN Data Channel Offload for NordVPN`: `103.86.96.100`, `103.86.99.100`
- Several virtual interfaces reported no IPv4 DNS servers or only placeholder IPv6 resolver entries

### Interface summary
- Material active interfaces for operator reasoning: `Ethernet`, `vEthernet (WSL (Hyper-V firewall))`, `NordLynx`, `Tailscale`
- Loopback and capture-only interfaces such as `Npcap Loopback Adapter` are present but not operationally primary

## 3) WSL and Container Platform
### WSL state
- Default distribution: `trolltyggr`
- Default WSL version: `2`
- Running distros at capture: `trolltyggr`, `docker-desktop`
- Additional distro present but stopped: `Ubuntu-24.04`
- `trolltyggr` distro OS: `Ubuntu 24.04.4 LTS (Noble Numbat)`
- Current `trolltyggr` IPv4 from `hostname -I`: `172.21.54.153`
- `hostname -i` inside `trolltyggr` failed with `Temporary failure in name resolution`, so direct name-to-IP resolution inside the distro should not be assumed healthy

### Docker state
- Docker Desktop version: `4.52.0 (210994)`
- Docker Engine version: `29.0.1`
- Client OS/Arch: `windows/amd64`
- Engine OS/Arch: `linux/amd64`
- Active Docker context: `desktop-linux`
- Docker Desktop service: `com.docker.service` running, automatic start
- Docker info at capture: `12` images, `3` containers total, `2` running, engine kernel `6.6.87.2-microsoft-standard-WSL2`

## 4) SSH Client and Server Environment
### OpenSSH version and components
- Client version: `OpenSSH_for_Windows_9.5p2, LibreSSL 3.8.2`
- `ssh-agent` service: running, automatic
- `sshd` service: running, automatic
- Loaded keys observed from `ssh-add -l`: one RSA key and one ED25519 key
- `SSH_AUTH_SOCK` was empty in the PowerShell capture despite `ssh-agent` running

### Client config and effective policy
- User config file present: `C:\Users\me\.ssh\config`
- Known hosts file present: `C:\Users\me\.ssh\known_hosts`
- Effective `ssh -G localhost` surfaces:
  - `StrictHostKeyChecking ask`
  - `CheckHostIP no`
  - `identityfile ~/.ssh/id_ed25519`
  - `identityfile ~/.ssh/id_rsa`
  - `identityfile ~/.ssh/_god`
  - `UserKnownHostsFile C:\Users\me/.ssh/known_hosts C:\Users\me/.ssh/known_hosts2`
  - `GlobalKnownHostsFile __PROGRAMDATA__\ssh/ssh_known_hosts __PROGRAMDATA__\ssh/ssh_known_hosts2`

### Server config highlights
Observed from `C:\ProgramData\ssh\sshd_config`:
- `Port 22`
- `PubkeyAuthentication yes`
- `PasswordAuthentication yes`
- `AuthorizedKeysFile .ssh/authorized_keys`
- `Subsystem sftp sftp-server.exe`
- `Match Group administrators` present

## 5) Local SSH Alias Map (`C:\Users\me\.ssh\config`)
| Alias | HostName | User | Notable options |
|---|---|---|---|
| `adagio` | `10.4.10.15` | `me` | `IdentityFile ~/.ssh/id_ed25519`, `PreferredAuthentications publickey`, `PasswordAuthentication no` |
| `adagio-pwsh` | `not present in current config` | `n/a` | `Legacy alias not currently defined on this host` |
| `ssh` | `10.4.10.15` | `sshuser` | `IdentityFile ~/.ssh/id_ed25519` |
| `heim` | `heimdall.local` | `luke` | `IdentityFile ~/.ssh/id_ed25519` |
| `largo` | `largo.local` | `luke` | `IdentityFile ~/.ssh/id_ed25519` |
| `tyggr` | `172.21.54.153` | `luke` | `HostKeyAlias tyggr`, `IdentityFile ~/.ssh/id_ed25519`, `PreferredAuthentications publickey`, `PasswordAuthentication no` |
| `tyggr-wsl` | `not present in current config` | `n/a` | `Windows-side profile currently uses direct `tyggr` alias instead` |

## 6) Current Interoperability Notes
- `ssh largo "echo OK_FROM_ADAGIO"` succeeded.
- `ssh heim "echo OK_FROM_ADAGIO"` timed out against `heimdall.local:22`.
- `ssh tyggr "echo OK_FROM_ADAGIO"` succeeded.
- Current Windows-side access to `tyggr` is functioning through the distro IP `172.21.54.153` rather than through a localhost-forwarding abstraction.
- The host currently exposes both inbound Windows OpenSSH (`sshd`) and a working outbound SSH path to both `largo` and the WSL distro.

### 2026-05-10 Cloudflare Tunnel and Access Delta
- A persistent Windows `Cloudflared` service is active under `LocalSystem`:
  - service name: `Cloudflared`
  - binary path: `C:\cloudflared\cloudflared.exe tunnel run --token ...`
  - live tunnel ID: `9c40f37a-e803-4973-bd74-91b8b9c3ed52`
- The live shared tunnel on `adagio` now carries these additional ingress rules:
  - `ssh-adagio.braisenly.com` -> `ssh://localhost:22`
  - `opencode-adagio.braisenly.com` -> `http://127.0.0.1:4096`
- New proxied DNS records were created for both `-adagio` hostnames and both public probes returned `302` through Cloudflare Access after cut-in.
- `OpenCode Web` is a user scheduled task for `me` that runs:
  - `powershell.exe -NoProfile -ExecutionPolicy Bypass -File "C:\Users\me\.cloudflared\start-opencode-web.ps1"`
- The local `opencode` origin on `adagio` was verified healthy on `http://127.0.0.1:4096/` after rewriting the task script to allow both:
  - `https://opencode.braisenly.com`
  - `https://opencode-adagio.braisenly.com`
- New Access apps were created with 30-day sessions:
  - `ssh-adagio.braisenly.com` (`ssh`, `720h`) with app-local allow policy for `me@lukezautke.com` constrained to SSH username `me`
  - `opencode-adagio.braisenly.com` (`self_hosted`, `720h`) with app-local allow policy for `me@lukezautke.com`

## 7) `sysinfo-up` Description
`adagio` is a Dell Inspiron 16 7610 running Windows 11 Home on build `26200`, with 32 GB of RAM and an 11th Gen Intel i7-11800H. Its operational primary identity is the LAN address `10.4.10.15` on `Ethernet`, but it also carries a WSL virtual-switch address (`172.21.48.1`), a NordLynx VPN address (`10.5.0.2`), and a Tailscale overlay address (`100.114.144.54`), so routing context matters whenever network behavior looks inconsistent.

This host is materially a Windows-plus-WSL machine rather than a plain Windows endpoint. Its default WSL distro is `trolltyggr` on WSL2, and the live distro address at capture time was `172.21.54.153`. Docker Desktop is active and backed by the WSL2 kernel, with the client operating from Windows and the engine running in Linux under the `desktop-linux` context. That makes `adagio` the control plane for both native Windows operations and Linux-container operations.

From an SSH standpoint, `adagio` is both a client and a server. Windows OpenSSH client and `sshd` are installed and running, `PubkeyAuthentication` is enabled, and `PasswordAuthentication` remains enabled server-side. The host can reach `largo` and `tyggr` successfully, while `heimdall` timed out during bounded probe testing. Future agents should treat `adagio` as a mixed-surface machine where Windows networking, WSL networking, SSH policy, and Docker Desktop state all interact directly with one another.

## 8) Evidence Registry
| Date (UTC offset surface) | Context | Concrete example | Source |
|---|---|---|---|
| 2026-03-16T23:05:17-06:00 | Host identity capture | `ADAGIO`, `adagio\me`, `Microsoft Windows 11 Home`, `Inspiron 16 7610`, `1.29.0`, `31.74 GiB RAM` | Local PowerShell via `Get-ComputerInfo`, CIM, and `systeminfo` |
| 2026-03-16T23:05:17-06:00 | Network snapshot capture | `Ethernet 10.4.10.15`, default route `10.4.10.1`, `vEthernet (WSL...) 172.21.48.1`, DNS `1.1.1.1/1.0.0.1` | `Get-NetIPAddress`, `Get-NetIPConfiguration`, `Get-DnsClientServerAddress`, `route print -4` |
| 2026-03-16T23:05:17-06:00 | SSH environment capture | `OpenSSH_for_Windows_9.5p2`, `ssh-agent` and `sshd` both running, loaded RSA + ED25519 keys, server `Port 22` and `Subsystem sftp-server.exe` | `ssh -V`, `ssh -Q key`, `ssh -G localhost`, `Get-Service`, `ssh-add -l`, `sshd_config` inspection |
| 2026-03-16T23:05:17-06:00 | WSL and container-platform capture | default distro `trolltyggr`, distro IP `172.21.54.153`, Docker Desktop `4.52.0`, Engine `29.0.1` | `wsl --status`, `wsl -l -v`, `wsl ... /etc/os-release`, `wsl ... hostname -I`, `docker version`, `docker info` |
| 2026-03-16T23:05:17-06:00 | Alias inventory capture | `adagio`, `ssh`, `heim`, `largo`, and `tyggr` present; `adagio-pwsh` absent | `C:\Users\me\.ssh\config` inspection |
| 2026-03-16T23:05:17-06:00 | Interop probe capture | `largo` OK, `heimdall.local` timed out, `tyggr` OK | bounded SSH probes from `adagio` |
| 2026-05-10T20:33:18-06:00 | Cloudflare `-adagio` browser endpoint activation | `ssh-adagio.braisenly.com` and `opencode-adagio.braisenly.com` returned `302` through Cloudflare Access; local `http://127.0.0.1:4096/` returned `200`; shared tunnel `9c40f37a-e803-4973-bd74-91b8b9c3ed52` gained the new ingress rules | Cloudflare tunnel config API, Cloudflare Access app/policy API, remote scheduled-task restart on `adagio`, and public curl probes from `largo` |

## 9) Validation Checklist
- [x] Machine identity captured with hostname, user, OS, build, architecture, manufacturer, model, CPU, BIOS, and memory
- [x] Network snapshot captured with primary IPv4 identities, gateway, DNS, and route context
- [x] SSH client and server environment captured with client policy plus server config highlights
- [x] WSL and Docker subsystem state captured because they materially affect this host
- [x] Local alias map documented from live `~/.ssh/config`
- [x] Interoperability probes recorded from this host to major neighbors
- [x] `sysinfo-up` summary written from raw evidence rather than memory

## 10) Handoff
When this profile is refreshed in the future:
1. Re-run the command set defined in [[Machine Profile Collection and Curation Protocol]].
2. Compare WSL distro IP behavior against current networking mode assumptions before changing SSH guidance.
3. Re-check whether `PasswordAuthentication yes` remains intentionally enabled in `sshd_config`.
4. Append new evidence rows and evolution-log rows rather than replacing prior findings.

## Sources
- Microsoft Learn: `Get-ComputerInfo`
  - https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-computerinfo
- Microsoft Learn: `systeminfo`
  - https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/systeminfo
- Windows PowerShell docs via Context7: `Get-NetIPConfiguration`
  - https://github.com/microsoftdocs/windows-powershell-docs/blob/main/docset/winserver2025-ps/NetTCPIP/Get-NetIPConfiguration.md
- Windows PowerShell docs via Context7: `Get-DnsClientServerAddress`
  - https://github.com/microsoftdocs/windows-powershell-docs/blob/main/docset/winserver2025-ps/DnsClient/Get-DnsClientServerAddress.md
- Microsoft Learn: WSL networking
  - https://learn.microsoft.com/en-us/windows/wsl/networking
- Microsoft Learn: OpenSSH for Windows overview
  - https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh-overview

## Quality Signals
- The addition of WSL and Docker sections materially improves this profile over the old stub because those subsystems directly affect registry, SSH, and networking behavior on `adagio`.
- The `Get-ComputerInfo` vs `systeminfo` product-name mismatch is exactly the type of cross-surface inconsistency future agents need recorded rather than normalized away.

### Quality Signal Log
| Date | Incident/Use | Most Useful Section(s) | Gap Found | Outcome Signal | Source |
|---|---|---|---|---|---|
| 2026-03-16 | Stub promotion to full host profile | `2) Network Snapshot`, `3) WSL and Container Platform`, `4) SSH Client and Server Environment`, `7) sysinfo-up Description` | No canonical machine-profile protocol note existed before this pass | `adagio` is now a real machine profile rather than a placeholder | Curator remediation pass on host-local evidence |
| 2026-05-10 | Cloudflare `-adagio` endpoint activation for Windows browser access | `6) Current Interoperability Notes`, `2026-05-10 Cloudflare Tunnel and Access Delta`, `8) Evidence Registry` | Prior machine profile had no authoritative record of the live Cloudflare tunnel service, the `OpenCode Web` scheduled task, or the new Access-protected `-adagio` hostnames | Future agents can recover browser access to `adagio` without rediscovering the Windows tunnel/task split | Live Cloudflare and host repair session |

## Evolution Log
| Date | Change | Reason | Trigger |
|---|---|---|---|
| 2026-02-18 | Created original `adagio` stub note and on-host runbook skeleton | A completed host profile did not yet exist for `adagio` | User request for Curator-Infra machine profiling |
| 2026-03-16 | Replaced the stub with a full host profile, added subsystem sections for WSL and Docker, and added a `sysinfo-up` summary backed by official Microsoft docs plus host-local evidence | The machine had become materially more important as a Windows/WSL/Docker control plane and the old stub no longer met KB standards | User request to fill the full `adagio` system profile and create or improve the machine-profile protocol |
| 2026-05-10 | Added the Cloudflare service/task split, the new `ssh-adagio` and `opencode-adagio` hostnames, and the Access-session details for the browser-facing Windows path | The host gained a live browser-access surface that future repairs need to treat as first-class infrastructure rather than tribal knowledge | User request to stand up `ssh-adagio.braisenly.com` and `opencode-adagio.braisenly.com` on the Windows box |
| 2026-05-28 | Added inbound relations to two previously-orphaned adagio operational-session notes (Docker Context Switch Repair 2026-03-19, Registry Login DNS Repair 2026-03-24) | Both session notes had no inbound link from any hub; this profile is their nearest canonical adagio hub | KB curation hygiene sweep (operations/reference partition) |

## Relations
- related_to [[SSH Recovery Snapshot - largo and adagio (2026-04-28)]]
- related_to [[Machine Profile - largo]]
- related_to [[Machine Profile Collection and Curation Protocol]]
- related_to [[SSH Remote Access Recovery and Operations Manual (Windows/Linux/macOS)]]
- related_to [[Local Network Docker Registry on adagio]]
- related_to [[Docker Registry Helper Commands on adagio and tyggr]]
- related_to [[Operational Session - Local Docker Registry Bootstrap and Registry Auth Repair on adagio and tyggr (2026-03-16)]]
- related_to [[Operational Session - adagio Docker Context Switch Repair (2026-03-19)]]
- related_to [[Operational Session - adagio Registry Login DNS Repair (2026-03-24)]]
- related_to [[Index – Agent KB]]
