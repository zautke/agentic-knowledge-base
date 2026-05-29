---
title: Operational Session - adagio Registry Login DNS Repair (2026-03-24)
type: note
entity_type: note
permalink: kb/operations/machines/operational-session-adagio-registry-login-dns-repair-2026-03-24
created: 2026-03-24
modified: 2026-05-28
status: draft
tags:
- project/agent-kb
- domain/docker
- domain/registry
- domain/dns
- machine/adagio
- status/draft
---

# Operational Session - adagio Registry Login DNS Repair (2026-03-24)

## User Request
Diagnose and patch failing `docker login adagio:5443` with web search, Context7, and remote verification on `adagio`.

## Initial Error
```text
docker login adagio:5443
Username: sshuser
Password:
Error response from daemon: Get "https://adagio:5443/v2/": dial tcp: lookup adagio on 0.250.250.200:53: no such host
```

## Initial State
- Local repo: `/Volumes/MACDEV/docker`
- Local shell helper repos already involved from prior work: `/Users/luke/.bash`, `/Users/luke/.local/bin`
- Remote repo on `adagio`: `C:\Users\me\dev\docker`
- Local host `/etc/hosts` already maps `10.4.10.15 adagio.local adagio`
- Local host resolver (`dscacheutil`) resolves `adagio` to `10.4.10.15`
- Local host `curl -skI https://adagio:5443/v2/` returns `401`, proving registry reachability and TLS acceptance from the host
- Active Docker engine at time of failure diagnosis: OrbStack (`docker context`/client context `orbstack`)
- Therefore the failing lookup path appears to be daemon-side DNS inside the engine VM, not host-side name resolution

## Hypothesis
`docker login` is reaching registry auth via the engine/VM network stack, which does not inherit the host `/etc/hosts` mapping for `adagio`. The documented IP fallback `10.4.10.15:5443` should work if the cert SAN and client CA trust are already in place for the IP endpoint.

## Immediate Next Steps
1. Re-read canonical registry notes.
2. Verify remote registry state from `adagio` itself.
3. Test daemon behavior against hostname vs IP.
4. Patch local helper/docs or client trust config as needed.

## Final Findings
- The registry on `adagio` was healthy the entire time. Remote verification confirmed `lan-registry` listening on `0.0.0.0:5443`, and local `docker login` attempts on `adagio` itself reached the registry and returned normal auth failures with a placeholder password.
- The user-visible failure `lookup adagio on 0.250.250.200:53: no such host` was client-side daemon DNS inside the active local OrbStack engine, not a server-side registry problem.
- The local macOS host resolver already knew `adagio -> 10.4.10.15`, and `curl -skI https://adagio:5443/v2/` from the host returned `401`. The mismatch was specifically between host DNS and daemon DNS.
- After switching to the documented IP fallback `10.4.10.15:5443`, the remaining issue was daemon CA trust. Once the registry CA was installed into `~/.docker/certs.d/adagio:5443/ca.crt` and `~/.docker/certs.d/10.4.10.15:5443/ca.crt`, daemon access to the IP endpoint progressed normally and authenticated successfully.

## Durable Patches
- Local repo `/Volumes/MACDEV/docker`
  - Branch: `fix/adagio-context-switch-20260319`
  - Commit: `cb66c5b`
  - Change: documented the exact daemon DNS failure and the supported IP fallback in `docs/local-private-registry-manual.md`.
- Local repo `/Users/luke/.bash`
  - Branch: `fix/adagio-registry-login-dns-20260324`
  - Commit: `94520ca`
  - Change: updated `.dockerrc` so registry helpers can read the real remote `registry/.env` over SSH, fetch the CA over SSH when the local `/c/...` path is absent, and automatically retry the IP endpoint when daemon-side lookup of `adagio` fails.

## Validation
- `docker login adagio:5443` on the local machine still fails under OrbStack with `lookup adagio on 0.250.250.200:53: no such host`; this is the unresolved daemon-DNS limitation.
- `docker login 10.4.10.15:5443 -u registryadmin --password-stdin` succeeded locally after CA installation.
- `docker pull 10.4.10.15:5443/registry:3` succeeded locally.

## Operational Outcome
- Supported working endpoint from this Mac under OrbStack: `10.4.10.15:5443`
- Helper-based working path after patch: use `rgca` to install trust and `rgli` to login; the helper will fall back to the IP endpoint automatically when the daemon cannot resolve `adagio`.
