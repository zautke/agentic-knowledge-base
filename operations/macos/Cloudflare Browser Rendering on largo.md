---
title: Cloudflare Browser Rendering on largo
type: reference
entity_type: reference
permalink: agent-kb/operations/macos/cloudflare-browser-rendering-on-largo
created: 2026-04-21
modified: 2026-05-28
status: evergreen
tags:
- project/agent-kb
- domain/cloudflare
- domain/browser-rendering
- domain/ssh
- domain/macos
- machine/largo
- status/evergreen
---

# Cloudflare Browser Rendering on largo

## Purpose and Scope
Canonical reference for the browser-rendered terminal workflow that reaches `largo` through Cloudflare Tunnel and is used for remote terminal work from an iPad browser. In local KB usage, `cloudflare browser rendering` names the actual deployed browser-rendered terminal path on this machine, not the generic Cloudflare developer product category.

## What This Entity Covers
This entity covers the host-side service path that makes the remote browser terminal usable:
- public ingress through Cloudflare Tunnel
- root `cloudflared` process ownership and restart behavior
- local SSH reachability to `localhost:22`
- OpenSSH keepalive policy on both client and server side
- local verification and recovery scripts stored in `~/.ssh/`
- the related local HTTP app exposed at `opencode.braisenly.com`
- hostname aliasing for `ssh.largo.braisenly.com` and `opencode.largo.braisenly.com`

It does not attempt to document every browser-side UI implementation detail. The canonical focus is the transport, host, and operations plane that keeps the session stable.

## Current Topology (2026-04-29)
1. An iPad browser reaches the public Cloudflare edge.
2. Cloudflare routes traffic to the root tunnel on `largo`.
3. The root tunnel is run by LaunchDaemon `com.cloudflare.cloudflared` using `/etc/cloudflared/config.yml`.
4. SSH traffic for `ssh.braisenly.com` and `ssh.largo.braisenly.com` is proxied to `ssh://localhost:22`.
5. The local OpenSSH daemon on `largo` is the macOS socket-activated daemon `com.openssh.sshd`, which reads `/etc/ssh/sshd_config` plus `/etc/ssh/sshd_config.d/*.conf`.
6. The interactive shell workload ultimately lands in the local terminal environment used for `zellij` work.
7. A sibling HTTP ingress for `opencode.braisenly.com` and `opencode.largo.braisenly.com` is routed through the same root tunnel to `http://127.0.0.1:4096`.

## Authoritative Local Artifacts
### Root tunnel config
- `/etc/cloudflared/config.yml`
- tunnel UUID: `3227fecc-8f59-41e4-9501-ab8235bc42f3`
- metrics endpoint: `127.0.0.1:9090/metrics`
- ingress hostnames observed on 2026-04-29:
  - `ssh.braisenly.com` -> `ssh://localhost:22`
  - `ssh.largo.braisenly.com` -> `ssh://localhost:22`
  - `opencode.braisenly.com` -> `http://127.0.0.1:4096`
  - `opencode.largo.braisenly.com` -> `http://127.0.0.1:4096`

Certificate caveat:
- `*.braisenly.com` does not automatically cover `*.*.braisenly.com`. If `*.largo.braisenly.com` names return TLS handshake failures, provision an advanced certificate pack for that wildcard (or use one-label hostnames such as `ssh-largo.braisenly.com`).

### Root process manager
- `/Library/LaunchDaemons/com.cloudflare.cloudflared.plist`
- effective `ProgramArguments` include:
  - `tunnel`
  - `--retries 10`
  - `--config /etc/cloudflared/config.yml`
  - `run`
  - `--protocol http2`
  - `--dns-resolver-addrs 1.1.1.1:53`
  - `--dns-resolver-addrs 1.0.0.1:53`

### SSH policy surfaces
- client policy: `~/.ssh/config`
- server policy: `/etc/ssh/sshd_config` plus `/etc/ssh/sshd_config.d/*.conf`
- effective stability values:
  - client: `ServerAliveInterval 30`, `ServerAliveCountMax 6`, `TCPKeepAlive no`
  - server: `ClientAliveInterval 30`, `ClientAliveCountMax 6`, `TCPKeepAlive no`

### Local helper scripts
- `~/.ssh/apply-root-terminal-stability.sh` - installs and restarts the authoritative root-side config.
- `~/.ssh/diagnose-terminal-stack.sh` - end-to-end verifier for files, process args, metrics, logs, and effective SSH config.

## Stability Baseline Chosen for This Host
### Cloudflare-side choices
- `protocol: http2` at process launch rather than QUIC
- `--retries 10`
- resolver pinning to `1.1.1.1:53` and `1.0.0.1:53`
- `originRequest.keepAliveTimeout: 300s`
- `originRequest.keepAliveConnections: 32`
- `originRequest.tcpKeepAlive: 30s`
- `no-autoupdate: true`

### SSH-side choices
- protocol keepalives are preferred over TCP keepalives
- both client and server use 30-second keepalive intervals with count max 6
- raw TCP keepalive is disabled on both sides to avoid relying on weaker spoofable keepalive semantics

## Why These Choices Became Canonical
### Observed failure shape before repair
The root tunnel logs showed repeated QUIC idle failures and resolver trouble before the protocol switch:
- `failed to accept QUIC stream: timeout: no recent network activity`
- `Failed to refresh DNS local resolver error="lookup region1.v2.argotunnel.com: i/o timeout"`

### Observed healthy state after repair
After moving the root tunnel to `http2`, pinning resolver addresses, and normalizing SSH keepalives, the authoritative log block showed:
- `Initial protocol http2`
- four registered tunnel connections using `protocol=http2`
- metrics serving on `127.0.0.1:9090/metrics`
- `localhost:22` accepting TCP

Inference:
- On this local network, UDP/QUIC path quality was worse than TCP/HTTP2 path quality for this long-lived SSH/browser terminal use case.
- Browser-rendered terminal stability depends on both tunnel transport and SSH application-layer keepalives; either side alone is not enough.

## Operational Boundaries
- The authoritative browser-rendering path is root-owned. User-scoped Homebrew `cloudflared` state is adjacent but not the source of truth for this workflow.
- The active `sshd` on this machine is macOS socket-activated `com.openssh.sshd`. Editing `/opt/homebrew/etc/ssh/sshd_config` does not repair the live daemon behavior for `localhost:22`.
- This note assumes a locked-down local network where operational stability is prioritized over hardening. Future public-Internet exposure changes would require a different security stance.
- For user `derp`, the `opencode` origin is now restored by a user LaunchAgent rather than a root service:
  - wrapper: `/Users/derp/.cloudflared/start_opencode_web.sh`
  - LaunchAgent: `/Users/derp/Library/LaunchAgents/com.braisenly.opencode.web.plist`
  - env source: `~/.bash/.env` (symlinked to the Obsidian-hosted shared env file)
- As of 2026-05-10, Cloudflare Access reusable policy `8c3a1652-0e9e-4c5f-b7b7-b0bc3451b665` constrains SSH usernames to `derp` for the browser-rendered SSH path.
- As of 2026-05-10, the SSH Access apps `ssh.braisenly.com` and `largo.braisenly.com/ssh` use `session_duration: 720h` (30 days) at the app level, and the app-local `Allow derp browser ssh` policies also use `session_duration: 720h`.
- The shared reusable policy remains at `24h`; it was intentionally left unchanged because it is also attached to non-SSH surfaces.

## Acceptance Criteria for Healthy State
A healthy browser-rendering stack on `largo` should satisfy all of the following:
1. `~/.ssh/diagnose-terminal-stack.sh` returns `RESULT PASS`.
2. Root `cloudflared` process is running with `--protocol http2` and resolver pinning.
3. Log block after last tunnel start shows `Initial protocol http2` and four registered edge connections.
4. `sshd -T -f /etc/ssh/sshd_config` shows the expected `ClientAlive*` and `TCPKeepAlive` values.
5. `ssh -G <host>` on the client side shows the expected `ServerAlive*` and `TCPKeepAlive` values.
6. `localhost:22` accepts TCP.
7. Metrics on `127.0.0.1:9090/metrics` respond.
8. When `opencode.braisenly.com` is part of the expected path, `127.0.0.1:4096` has a live local origin.

## Failure Modes That Matter Most
- QUIC idle timeout and stream acceptance failures on the root tunnel.
- Resolver instability when `cloudflared` relies on a less reliable local DNS path.
- Editing the wrong `sshd` config tree (`/opt/homebrew/etc/ssh/...` instead of `/etc/ssh/...`).
- Repairing the user LaunchAgent while the real production path is the root LaunchDaemon.
- Assuming a config file change is live without checking effective config and live process arguments.
- Treating a dead local `opencode` origin as a Cloudflare problem when the tunnel plane is already healthy.

## 2026-05-10 Base-Hostname Recovery Delta
- Recovery scope was reduced to the base hostnames:
  - `ssh.braisenly.com`
  - `opencode.braisenly.com`
- Tunnel control plane was already healthy; no Cloudflare tunnel recreation was required.
- The actual outage component for browser-rendering support was a missing local `opencode` listener on `127.0.0.1:4096`.
- `opencode web` was validated locally and then made persistent for user `derp` with a LaunchAgent.
- Verification after recovery:
  - LaunchAgent running under `gui/503`
  - local `127.0.0.1:4096` returned `200`
  - public `ssh.braisenly.com` and `opencode.braisenly.com` both returned `302` through Cloudflare Access

## 2026-05-10 Browser SSH Short-Lived Cert Delta
- Added app-local Access allow policies for `derp@braisenly.com` on the SSH apps instead of modifying the shared reusable policy across unrelated apps.
- Created short-lived SSH CAs for:
  - `ssh.braisenly.com`
  - `largo.braisenly.com/ssh`
- Cloudflare-side prep is complete for browser/native short-lived certificate authentication.
- Remaining host-side requirement is root-owned OpenSSH trust configuration under `/etc/ssh`:
  - `TrustedUserCAKeys /etc/ssh/ca.pub`
  - any required principal mapping if the Unix username and Access identity prefix differ
- Important browser limitation from Cloudflare docs:
  - browser-rendered short-lived certificate auth expects the user's email prefix to match the Unix username
  - with local Unix user `derp`, the clean browser-cert identity is `derp@...`

## Authoritative Basis
### Local evidence
- `/Library/Logs/cloudflare/cloudflared.err.log`
- `/etc/cloudflared/config.yml`
- `/Library/LaunchDaemons/com.cloudflare.cloudflared.plist`
- `~/.ssh/config`
- `/etc/ssh/sshd_config`
- `/etc/ssh/sshd_config.d/`
- `~/.ssh/diagnose-terminal-stack.sh`
- `~/.ssh/apply-root-terminal-stability.sh`

### Official references consulted
- Cloudflare Tunnel common errors: https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/troubleshoot-tunnels/common-errors/
- Cloudflare Tunnel firewall/protocol requirements: https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/configure-tunnels/tunnel-with-firewall/
- OpenBSD `ssh_config(5)`: https://man.openbsd.org/ssh_config
- OpenBSD `sshd_config(5)`: https://man.openbsd.org/sshd_config

## Relations
- related_to [[Index - Cloudflare Browser Rendering Operations]]
- related_to [[Runbook - Cloudflare Browser Rendering Terminal Verification and Recovery on largo]]
- related_to [[Operational Session - Cloudflare Browser Rendering Terminal Stabilization on largo (2026-04-21)]]
- related_to [[Machine Profile - largo]]
- related_to [[SSH Remote Access Recovery and Operations Manual (Windows/Linux/macOS)]]
- related_to [[Codex MCP Smoke 2026-03-13 23-05-30]]
- related_to [[Auggie Supergateway Streamable HTTP on Port 7440 (macOS)]]
- related_to [[Index – Agent KB]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-05-10 | Added user-`derp` LaunchAgent recovery details for the base hostnames and documented the local `opencode` origin as the actual missing component. | Preserve the real fix path so future agents do not waste time rebuilding a healthy Cloudflare tunnel plane. | User request to bring the `ssh` and `opencode` tunnels back up for a new user on the same machine. |
| 2026-05-10 | Corrected the authoritative `sshd` path to macOS socket-activated `com.openssh.sshd` and recorded the Cloudflare Access username constraint for `derp`. | Browser SSH auth debugging showed the live service was not the Homebrew daemon and that Access policy now carries the username mapping. | User report that browser-rendered SSH showed the wrong user and still offered a private-key path. |
| 2026-04-29 | Added `ssh.largo.braisenly.com` and `opencode.largo.braisenly.com` as tunnel ingress aliases for the same local origins; documented nested-subdomain TLS certificate caveat. | New hostname cutover required preserving existing hostnames while adding machine-scoped aliases. | User request to bring `ssh.largo.braisenly.com` and `opencode.largo.braisenly.com` online on this machine. |
| 2026-04-21 | Created canonical entity note for the browser-rendered terminal stack on `largo`, including topology, authoritative local artifacts, and current stabilization choices. | The KB previously had only adjacent SSH and Cloudflare notes, not a primary entity for this deployed workflow. | User request to make this agent the primary curator for the new `cloudflare browser rendering` entity. |
