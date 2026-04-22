---
title: Operational Session - Cloudflare Browser Rendering Terminal Stabilization on
  largo (2026-04-21)
type: reference
permalink: agent-kb/operations/machines/operational-session-cloudflare-browser-rendering-terminal-stabilization-on-largo-2026-04-21
tags:
- project/agent-kb
- domain/cloudflare
- domain/browser-rendering
- domain/ssh
- domain/macos
- machine/largo
- status/draft
---

# Operational Session - Cloudflare Browser Rendering Terminal Stabilization on largo (2026-04-21)

## Purpose and Scope
Dated execution record for the repair session that stabilized the browser-rendered terminal path back to `largo` for iPad-based terminal work. This session covered research, root-cause isolation, root tunnel normalization, SSH keepalive normalization, launchd repair, verifier creation, and final acceptance checks.

## Session Trigger
User-reported problem:
- browser-rendered terminal sessions routed back to this machine were dropping
- the path was used for `zellij` terminal work from an iPad
- the user wanted globally applied keepalive and retry settings within a locked-down local network where stability mattered more than hardening

## Starting Hypothesis
Potential failure domains at session start:
1. Cloudflare tunnel transport instability
2. local DNS/resolver instability affecting `cloudflared`
3. SSH idle timeout behavior
4. editing the wrong control plane for the real production path

## Local Evidence Collected
### Effective SSH client state
`ssh -G` confirmed the intended client keepalive values after user config repair:
- `tcpkeepalive no`
- `serveraliveinterval 30`
- `serveralivecountmax 6`

### Effective SSH server state
The active `sshd` process on this host was Homebrew OpenSSH, not the Apple system daemon. Effective server values after repair:
- `tcpkeepalive no`
- `clientaliveinterval 30`
- `clientalivecountmax 6`

### Root tunnel config and process state
Observed authoritative root artifacts:
- `/etc/cloudflared/config.yml`
- `/Library/LaunchDaemons/com.cloudflare.cloudflared.plist`
- root process args included `--protocol http2`, `--retries 10`, and explicit DNS resolver addresses

### Root tunnel failure signatures before repair
`/Library/Logs/cloudflare/cloudflared.err.log` showed repeated QUIC-path failures:
- `failed to accept QUIC stream: timeout: no recent network activity`
- `failed to run the datagram handler error="timeout: no recent network activity"`
- `Failed to refresh DNS local resolver error="lookup region1.v2.argotunnel.com: i/o timeout"`

Inference:
- QUIC over UDP was unstable on this path.
- local resolver behavior was also unreliable enough to justify explicit resolver pinning.

## External Research Used
### Cloudflare official guidance
Cloudflare Tunnel docs state:
- Tunnel uses TCP for `http2` and UDP for `quic` on port `7844`.
- idle SSH and other long-lived sessions may disconnect more often when QUIC/UDP traffic is aggressively timed out by network devices.
- Cloudflare recommends testing with `protocol: http2` and configuring application-layer keepalives such as SSH `ServerAliveInterval` when those idle disconnects happen.

### OpenSSH official guidance
OpenBSD manual pages state:
- `ServerAliveInterval` and `ServerAliveCountMax` are protocol-level keepalives sent on the encrypted channel.
- `TCPKeepAlive` is different and spoofable.
- server-side `ClientAliveInterval` and `ClientAliveCountMax` provide the parallel daemon-side behavior.

## Actions Performed
### 1. Normalized SSH client policy
Updated `~/.ssh/config` to use:
- `TCPKeepAlive no`
- `ServerAliveInterval 30`
- `ServerAliveCountMax 6`

### 2. Built root Cloudflare config and LaunchDaemon recommendations
Prepared:
- `~/.cloudflared/root-cloudflared-config.recommended.yml`
- `~/.ssh/com.cloudflare.cloudflared.plist.recommended`

Key root tunnel decisions:
- `protocol http2`
- `retries 10`
- DNS resolvers pinned to `1.1.1.1:53` and `1.0.0.1:53`
- `keepAliveTimeout 300s`
- `keepAliveConnections 32`
- `tcpKeepAlive 30s`
- metrics on `127.0.0.1:9090`

### 3. Wrote root repair script
Created `~/.ssh/apply-root-terminal-stability.sh` to install root-side config and restart the services.

### 4. Hit two real execution bugs and repaired them
First bug:
- used the wrong `cloudflared` validation syntax
- incorrect form: `cloudflared tunnel ingress validate --config ...`
- corrected form: `cloudflared tunnel --config FILE ingress validate`

Second bug:
- initial server-side fix targeted `/etc/ssh/sshd_config.d/*`
- live daemon actually used `/opt/homebrew/etc/ssh/sshd_config`
- script was corrected to patch the Homebrew config and validate with `sshd -t -f /opt/homebrew/etc/ssh/sshd_config`

### 5. User executed corrected root repair script
After the corrected run:
- root `cloudflared` started successfully with `protocol http2`
- root tunnel registered four connections over `http2`
- `sshd` effective keepalive settings matched the intended baseline

### 6. Wrote reusable verifier
Created `~/.ssh/diagnose-terminal-stack.sh`.

Verifier responsibilities:
- confirm required files exist
- inspect effective client/server SSH config
- inspect root Cloudflare config content and LaunchDaemon content
- confirm `localhost:22`
- confirm live process args
- confirm metrics on `127.0.0.1:9090`
- confirm log block after last root tunnel start shows `http2` and four edge connections

## Final Verified State
Observed after the successful repair:
- root process running: `cloudflared tunnel --retries 10 --config /etc/cloudflared/config.yml run --protocol http2 --dns-resolver-addrs 1.1.1.1:53 --dns-resolver-addrs 1.0.0.1:53`
- root log block showed `Initial protocol http2`
- root log block showed four `Registered tunnel connection` lines with `protocol=http2`
- `localhost:22` accepted TCP
- `127.0.0.1:9090/metrics` responded
- verifier result: `RESULT PASS failures=0 warnings=0`

## Residual Notes
- an unrelated user/token `cloudflared` process still existed separately with metrics on `127.0.0.1:9091`
- that user-scoped tunnel was not treated as authoritative for the browser-rendered terminal path

## What Future Agents Must Remember
- root `cloudflared` is production here
- Homebrew `sshd` path is production here
- effective config beats file assumptions
- if QUIC idle failures reappear, confirm the root service did not drift back off `http2`
- use the verifier before claiming success

## Sources
### Local evidence
- `/Library/Logs/cloudflare/cloudflared.err.log`
- `/etc/cloudflared/config.yml`
- `/Library/LaunchDaemons/com.cloudflare.cloudflared.plist`
- `~/.ssh/config`
- `/opt/homebrew/etc/ssh/sshd_config`
- `~/.ssh/apply-root-terminal-stability.sh`
- `~/.ssh/diagnose-terminal-stack.sh`

### Official references
- Cloudflare Tunnel common errors: https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/troubleshoot-tunnels/common-errors/
- Cloudflare Tunnel firewall/protocol requirements: https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/configure-tunnels/tunnel-with-firewall/
- OpenBSD `ssh_config(5)`: https://man.openbsd.org/ssh_config
- OpenBSD `sshd_config(5)`: https://man.openbsd.org/sshd_config

## Relations
- related_to [[Cloudflare Browser Rendering on largo]]
- related_to [[Runbook - Cloudflare Browser Rendering Terminal Verification and Recovery on largo]]
- related_to [[Machine Profile - largo]]
- related_to [[SSH Remote Access Recovery and Operations Manual (Windows/Linux/macOS)]]
- related_to [[Index - Cloudflare Browser Rendering Operations]]
- related_to [[Codex MCP Smoke 2026-03-13 23-05-30]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-21 | Created dated operational record for the browser-rendered terminal stabilization session. | Preserve the full repair chronology, evidence, mistakes, corrections, and acceptance signals instead of compressing them into a final-state note only. | User request to document execution, process, and procedures in detail. |