---
title: Machine Profile Collection and Curation Protocol
type: instruction
permalink: kb/instructions/machine-profile-collection-and-curation-protocol
tags:
- project/agent-kb
- domain/infrastructure
- domain/networking
- domain/ssh
- domain/system-inventory
- op/profile-machine
- status/evergreen
---

# Machine Profile Collection and Curation Protocol

## Purpose
Canonical workflow for creating or refreshing machine-profile notes from host-local evidence while preserving deterministic structure, source attribution, and operational usefulness.

## When to Use
- a machine profile does not exist
- a machine profile still exists only as a stub
- OS, network, SSH, WSL, Docker, or auth behavior changed materially
- an incident session generated machine-local evidence that should become durable host knowledge

## Required Outputs
A complete machine profile should contain:
- machine identity
- hardware and firmware summary
- network snapshot
- SSH client and server environment
- local alias map
- subsystem state that materially affects operations on that host
- interoperability notes from actual probes
- evidence registry
- `sysinfo-up` prose summary
- quality signals and evolution log

## Authoritative Basis
Use host-local commands first, then interpret them using official docs.

### Windows system inventory
- `Get-ComputerInfo` returns a consolidated object of system and operating-system properties.
- `systeminfo` displays detailed configuration information about a computer and its operating system, including OS configuration, security information, product ID, and hardware properties.

### Windows network inventory
- `Get-NetIPConfiguration` gets network configuration including usable interfaces, IP addresses, and DNS servers.
- `Get-DnsClientServerAddress` returns DNS server IP addresses per interface.
- `Get-NetIPAddress` returns current IP address configuration and interface association.

### WSL networking interpretation
- Microsoft documents WSL default NAT networking and recommends mirrored networking mode for newer feature behavior.
- In NAT mode, host and distro often need peer-IP lookup.
- In mirrored mode, host and WSL can often use `localhost`, but live host-local evidence should win over assumptions.

### Windows OpenSSH interpretation
- OpenSSH on Windows is a client-server architecture.
- `ssh` is the client, `sshd` the server, `ssh-agent` stores keys, and `scp`/`sftp` ride over SSH.

## Deterministic Collection Sequence
### Phase 0: Preflight
1. Read any existing machine profile.
2. Read related incident/session notes.
3. Read the nearest interoperability or recovery manual if one exists.
4. Build triage buckets before collecting evidence: `identity`, `network`, `ssh`, `subsystems`, `interop`, `risks`.

### Phase 1: Identity and hardware
Capture:
- UTC timestamp
- local timestamp
- hostname
- local username
- OS name and version
- kernel/build
- architecture
- manufacturer/model
- CPU
- memory
- BIOS/firmware version
- install date / last boot if relevant

Recommended Windows commands:
```powershell
Get-Date -Format o
Get-Date -Format "yyyy-MM-dd HH:mm:ss zzz"
$env:COMPUTERNAME
whoami
Get-ComputerInfo
Get-CimInstance Win32_OperatingSystem
Get-CimInstance Win32_ComputerSystem
Get-CimInstance Win32_BIOS
Get-CimInstance Win32_Processor
systeminfo
```

### Phase 2: Network snapshot
Capture:
- primary IPv4 addresses
- interface aliases and role
- default route and gateway
- DNS servers per interface
- VPN/tunnel or virtual interfaces that materially affect routing

Recommended Windows commands:
```powershell
Get-NetIPAddress -AddressFamily IPv4
Get-NetIPConfiguration
Get-DnsClientServerAddress -AddressFamily IPv4
route print -4
```

### Phase 3: SSH environment
Capture:
- `ssh -V`
- `ssh -Q key`
- effective `ssh -G localhost` or target-specific output for key policy and known-host handling
- `ssh-agent` and `sshd` service state
- loaded keys from `ssh-add -l`
- presence and timestamps of `~/.ssh/config`, `known_hosts`, and server config files
- material `sshd_config` directives such as `Port`, `PubkeyAuthentication`, `PasswordAuthentication`, `AuthorizedKeysFile`, and `Subsystem`

### Phase 4: Subsystem state
Capture any subsystem that materially changes machine behavior for future agents, such as:
- WSL default distro, version, and active distro inventory
- WSL distro OS identity and IP behavior
- Docker Desktop / Docker Engine version and context
- VPN or overlay-network tooling when it affects routing

Recommended Windows commands:
```powershell
wsl --status
wsl -l -v
wsl -d <DistroName> cat /etc/os-release
wsl -d <DistroName> hostname -I
docker version
docker info
Get-Service com.docker.service
```

### Phase 5: Alias map and interoperability probes
Document current `~/.ssh/config` aliases and run bounded probes to the important neighbors.

Recommended Windows commands:
```powershell
Get-Content $env:USERPROFILE\.ssh\config | Select-String -Pattern '^Host\s+(...)'
ssh -o BatchMode=yes -o ConnectTimeout=7 <alias> "echo OK_FROM_<HOST>"
```

For each failed probe, record the exact failure text and classify it as:
- transport
- authentication
- session/subsystem
- naming/resolution

### Phase 6: `sysinfo-up` summary
After raw evidence capture, write one concise prose section that describes the machine as an operator would actually use it. That section should summarize:
- what the machine is
- what OS/hardware it runs
- which network identities matter
- what local subsystem stack matters most
- what remote access posture exists
- what current risks or inconsistencies matter

This section is an interpretation layer, not a replacement for raw evidence.

## Note-Shaping Rules
- Prefer explicit tables for identity and alias maps.
- Keep evidence rows append-only.
- Keep evolution-log rows append-only.
- If two command surfaces disagree, record both and explain which one should be treated as primary.
- Do not erase historical incident evidence when refreshing the note.

## Anti-Patterns
- filling placeholders from memory instead of host-local commands
- reducing a machine profile to only hostname and IP
- omitting SSH server state on a machine that accepts inbound SSH
- omitting WSL or Docker state when those subsystems materially affect networking or operations
- writing a prose-only host description without preserving command-level evidence

## Sources
- Microsoft Learn: `Get-ComputerInfo`
  - https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-computerinfo
- Microsoft Learn: `systeminfo`
  - https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/systeminfo
- Windows PowerShell docs surfaced via Context7: `Get-NetIPConfiguration`
  - https://github.com/microsoftdocs/windows-powershell-docs/blob/main/docset/winserver2025-ps/NetTCPIP/Get-NetIPConfiguration.md
- Windows PowerShell docs surfaced via Context7: `Get-DnsClientServerAddress`
  - https://github.com/microsoftdocs/windows-powershell-docs/blob/main/docset/winserver2025-ps/DnsClient/Get-DnsClientServerAddress.md
- Microsoft Learn: WSL networking
  - https://learn.microsoft.com/en-us/windows/wsl/networking
- Microsoft Learn: OpenSSH for Windows overview
  - https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh-overview

## Relations
- related_to [[Machine Profile - largo]]
- related_to [[Machine Profile - adagio]]
- related_to [[Curator Session Primer and Runbook]]
- related_to [[Semantic Relationship Discovery and Linking Protocol]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-16 | Created canonical machine-profile collection protocol from the existing `largo` exemplar and `adagio` stub, then expanded it with Windows system-inventory, WSL, Docker, and OpenSSH guidance backed by official docs. | Machine profiling had patterns but no single protocol note, which made stub promotion and future host capture inconsistent. | User request to find or create the machine-profile protocol and improve it before filling `adagio`. |