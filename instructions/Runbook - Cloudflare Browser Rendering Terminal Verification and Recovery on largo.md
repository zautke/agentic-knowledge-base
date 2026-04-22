---
title: Runbook - Cloudflare Browser Rendering Terminal Verification and Recovery on
  largo
type: instruction
permalink: agent-kb/instructions/runbook-cloudflare-browser-rendering-terminal-verification-and-recovery-on-largo
tags:
- project/agent-kb
- domain/cloudflare
- domain/browser-rendering
- domain/ssh
- domain/macos
- machine/largo
- op/runbook
- status/evergreen
---

# Runbook - Cloudflare Browser Rendering Terminal Verification and Recovery on largo

## Purpose and Scope
Operator runbook for verifying, diagnosing, and repairing the browser-rendered terminal path on `largo` when sessions drop or the tunnel path appears unstable. This runbook is specific to the current host layout: root `cloudflared`, Homebrew `sshd`, local SSH origin on `localhost:22`, and the browser terminal use case over Cloudflare Tunnel.

## Definition of Healthy
Declare the path healthy only when all checks pass:
1. root `cloudflared` process is running with `--protocol http2`.
2. root tunnel log block after last start shows `Initial protocol http2` and four `Registered tunnel connection` lines with `protocol=http2`.
3. `localhost:22` accepts TCP.
4. client SSH effective config shows `ServerAliveInterval 30`, `ServerAliveCountMax 6`, `TCPKeepAlive no`.
5. server SSH effective config shows `ClientAliveInterval 30`, `ClientAliveCountMax 6`, `TCPKeepAlive no`.
6. `http://127.0.0.1:9090/metrics` responds.
7. `~/.ssh/diagnose-terminal-stack.sh` exits with `RESULT PASS`.

## Fast Path
If the system was already normalized and you only need to confirm health:
```bash
~/.ssh/diagnose-terminal-stack.sh
```
If that returns `RESULT PASS`, stop. Do not improvise further changes.

## Phase 0: Confirm Which Stack Owns Production
The most important distinction is root tunnel vs user tunnel.

Check live root process:
```bash
ps aux | rg '[c]loudflared tunnel --retries 10 --config /etc/cloudflared/config.yml run --protocol http2'
```
Check for unrelated user-scoped token tunnel:
```bash
ps aux | rg '[c]loudflared tunnel --metrics 127\.0\.0\.1:9091 run --token'
```

Interpretation:
- the root LaunchDaemon process is the authoritative browser-rendering path.
- the user/token tunnel may coexist, but it is not the source of truth for this workflow.

## Phase 1: Verify Static Files
```bash
test -f /etc/cloudflared/config.yml
test -f /Library/LaunchDaemons/com.cloudflare.cloudflared.plist
test -f /opt/homebrew/etc/ssh/sshd_config
test -f ~/.ssh/config
```

Required content checks:
```bash
rg -n 'service: ssh://localhost:22|keepAliveTimeout: 300s|keepAliveConnections: 32|tcpKeepAlive: 30s' /etc/cloudflared/config.yml
plutil -convert xml1 -o - /Library/LaunchDaemons/com.cloudflare.cloudflared.plist | rg -n -- '--protocol|http2|--retries|10|--dns-resolver-addrs|1.1.1.1:53|1.0.0.1:53'
```

Critical guardrail:
- active `sshd` config is `/opt/homebrew/etc/ssh/sshd_config`.
- do not assume `/etc/ssh/sshd_config` is live on this machine.

## Phase 2: Verify Effective SSH Policy
### Client
```bash
ssh -G adagio | rg '^(tcpkeepalive|serveraliveinterval|serveralivecountmax) '
```
Expected:
- `tcpkeepalive no`
- `serveraliveinterval 30`
- `serveralivecountmax 6`

### Server
```bash
sshd -T -f /opt/homebrew/etc/ssh/sshd_config | rg '^(tcpkeepalive|clientaliveinterval|clientalivecountmax) '
```
Expected:
- `tcpkeepalive no`
- `clientaliveinterval 30`
- `clientalivecountmax 6`

## Phase 3: Verify Tunnel Log Shape
Inspect recent root log block:
```bash
tail -n 80 /Library/Logs/cloudflare/cloudflared.err.log
```

Good signals:
- `Starting tunnel tunnelID=3227fecc-8f59-41e4-9501-ab8235bc42f3`
- `Initial protocol http2`
- four `Registered tunnel connection` lines with `protocol=http2`

Bad signals that matter on this host:
- `failed to accept QUIC stream: timeout: no recent network activity`
- `Failed to refresh DNS local resolver`
- repeated `ERR` lines after the last root tunnel start

Interpretation:
- QUIC-related idle stream failures strongly suggest the wrong transport for this network path.
- DNS refresh trouble suggests the resolver path is unstable; keep the explicit resolver arguments.

## Phase 4: Verify Origin Reachability and Metrics
```bash
nc -z localhost 22
curl -fsS http://127.0.0.1:9090/metrics | rg cloudflared_tunnel
```

Interpretation:
- if `localhost:22` fails, the problem is local SSH daemon or host firewall, not Cloudflare edge transport alone.
- if metrics fail, root `cloudflared` is likely not running correctly even if stale logs exist.

## Phase 5: Validate Config Before Restarting
Use the correct validation syntax:
```bash
cloudflared tunnel --config /Users/luke/.cloudflared/root-cloudflared-config.recommended.yml ingress validate
sudo sshd -t -f /opt/homebrew/etc/ssh/sshd_config
```

Guardrail:
- `cloudflared tunnel ingress validate --config ...` is wrong for this CLI form on this host.
- use `cloudflared tunnel --config FILE ingress validate`.

## Phase 6: Repair Procedure
When config drift or process drift is found, run:
```bash
~/.ssh/apply-root-terminal-stability.sh
```

What that script is expected to do:
- backup current root Cloudflare and sshd files
- install the canonical root Cloudflare config to `/etc/cloudflared/config.yml`
- install the canonical LaunchDaemon plist
- patch `/opt/homebrew/etc/ssh/sshd_config` with current keepalive policy
- validate `sshd` and `cloudflared` config
- restart `sshd`
- restart root `cloudflared`

## Phase 7: Post-Repair Verification
Immediately run:
```bash
~/.ssh/diagnose-terminal-stack.sh
```

Then manually spot-check:
```bash
ps aux | rg '[c]loudflared tunnel --retries 10 --config /etc/cloudflared/config.yml run --protocol http2'
sshd -T -f /opt/homebrew/etc/ssh/sshd_config | rg 'clientalive|tcpkeepalive'
ssh -G adagio | rg 'serveralive|tcpkeepalive'
tail -n 40 /Library/Logs/cloudflare/cloudflared.err.log
```

## Troubleshooting Branches
### Case A: root `cloudflared` is down
- inspect LaunchDaemon plist content
- inspect `/etc/cloudflared/config.yml`
- validate ingress config with the correct CLI syntax
- restart via `~/.ssh/apply-root-terminal-stability.sh`

### Case B: `cloudflared` is up but still on QUIC
- verify live process args, not only config files
- re-run root apply script
- confirm the root service was actually restarted

### Case C: SSH still drops but tunnel is healthy
- verify effective SSH keepalive values, not only file contents
- confirm Homebrew `sshd` config path
- verify `localhost:22` stays reachable during a test window

### Case D: logs show DNS timeout again
- confirm `--dns-resolver-addrs` is still present in the live root process args
- confirm local network resolver drift did not overwrite the root plist

## Anti-Patterns
| Anti-pattern | Why it is wrong | Correct behavior |
|---|---|---|
| Editing only `/etc/ssh/sshd_config` | On this host the active daemon reads `/opt/homebrew/etc/ssh/sshd_config` | Validate and patch the Homebrew config path |
| Repairing user LaunchAgent instead of root LaunchDaemon | Browser-rendering path uses the root tunnel | Check live root process args first |
| Trusting file contents without checking effective config | `ssh` and `sshd` may still run with different effective values | Use `ssh -G` and `sshd -T` |
| Leaving QUIC enabled after confirmed UDP idle failures | Reintroduces known instability on this network | keep root tunnel on `http2` until evidence says otherwise |
| Declaring success without verifier output | Easy to miss partial drift | require `~/.ssh/diagnose-terminal-stack.sh` to pass |

## Authoritative Basis
- local configs and logs on `largo`
- Cloudflare Tunnel common errors: https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/troubleshoot-tunnels/common-errors/
- Cloudflare Tunnel firewall/protocol requirements: https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/configure-tunnels/tunnel-with-firewall/
- OpenBSD `ssh_config(5)`: https://man.openbsd.org/ssh_config
- OpenBSD `sshd_config(5)`: https://man.openbsd.org/sshd_config

## Relations
- related_to [[Cloudflare Browser Rendering on largo]]
- related_to [[Operational Session - Cloudflare Browser Rendering Terminal Stabilization on largo (2026-04-21)]]
- related_to [[Machine Profile - largo]]
- related_to [[SSH Remote Access Recovery and Operations Manual (Windows/Linux/macOS)]]
- related_to [[Index - Cloudflare Browser Rendering Operations]]

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-21 | Created operator runbook for verifying and recovering the browser-rendered terminal path on `largo`. | The live repair produced repeatable procedures and guardrails that should not stay trapped inside shell history or one incident note. | User request to write detailed procedural notes from the completed execution. |