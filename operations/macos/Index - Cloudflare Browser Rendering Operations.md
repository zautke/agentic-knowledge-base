---
title: Index - Cloudflare Browser Rendering Operations
type: index
entity_type: index
permalink: agent-kb/operations/macos/index-cloudflare-browser-rendering-operations
created: 2026-04-21
modified: 2026-05-28
status: evergreen
tags:
- project/agent-kb
- domain/cloudflare
- domain/browser-rendering
- domain/ssh
- domain/macos
- index/cloudflare-browser-rendering
- status/evergreen
---

# Index - Cloudflare Browser Rendering Operations

## Purpose and Scope
Navigation hub for the browser-rendered terminal path on `largo` that is exposed through Cloudflare Tunnel and used for remote terminal work from an iPad browser. This hub exists to keep the operational model split into a canonical entity note, an execution runbook, and a dated session record instead of burying all of the logic inside `Machine Profile - largo` or the generic SSH recovery manual.

## Canonical Notes
- [[Cloudflare Browser Rendering on largo]] - canonical entity note for the deployed browser-rendered terminal path, its topology, authoritative config surfaces, and current hardening choices.
- [[Runbook - Cloudflare Browser Rendering Terminal Verification and Recovery on largo]] - operator procedure for verifying health, diagnosing drops, restarting the root tunnel, and validating the full path end-to-end.
- [[Operational Session - Cloudflare Browser Rendering Terminal Stabilization on largo (2026-04-21)]] - dated execution record for the QUIC-drop investigation, root service repair, keepalive normalization, and reusable verifier creation.
- [[Operational Session - Cloudflare Hostname Alias Cutover on largo (2026-04-29)]] - dated execution record for adding `ssh.largo.braisenly.com` and `opencode.largo.braisenly.com`, with TLS certificate-coverage findings for nested subdomains.
- [[Codex MCP Smoke 2026-03-13 23-05-30]] - earlier proof that Cloudflare Tunnel was already part of the local control plane before the browser-rendering terminal workflow was formalized.

## Related Infrastructure Context
- [[Machine Profile - largo]] - host baseline, current network shape, and SSH client profile for the machine that terminates this workflow.
- [[SSH Remote Access Recovery and Operations Manual (Windows/Linux/macOS)]] - broader SSH recovery doctrine; this Cloudflare cluster extends that manual for tunnel-mediated long-lived browser terminal sessions.
- [[Auggie Supergateway Streamable HTTP on Port 7440 (macOS)]] - adjacent macOS note showing the local pattern of wrapping internal services behind reusable HTTP-facing control-plane entrypoints.
- [[Cloudflare Access rollout and rollback protocol for tunnel hostnames]] - reusable rollout/rollback protocol for the Cloudflare Access apps that gate these tunnel hostnames.
- [[Index – Agent KB]] - top-level KB entry point.

## Canonical Stack Decisions (2026-04-21)
- The authoritative browser-rendering tunnel is the root `cloudflared` LaunchDaemon, not the user-scoped Homebrew LaunchAgent.
- The root tunnel is pinned to `protocol: http2` because the observed QUIC path on this network produced idle-stream failures and DNS refresh instability.
- SSH stability is enforced at the protocol layer on both sides: `ServerAliveInterval 30`, `ServerAliveCountMax 6`, `ClientAliveInterval 30`, `ClientAliveCountMax 6`, and `TCPKeepAlive no`.
- The active `sshd` config on this host is `/opt/homebrew/etc/ssh/sshd_config`, not `/etc/ssh/sshd_config`.
- The reusable acceptance check for this stack is `~/.ssh/diagnose-terminal-stack.sh`.

## Trigger Conditions for Update
- `cloudflared` transport changes again, especially between `quic` and `http2`.
- LaunchDaemon arguments, resolver behavior, metrics port, or ingress hostnames change.
- SSH keepalive policy changes on either client or server side.
- Browser-rendered terminal drops recur with a new failure signature.
- `opencode.braisenly.com` or `ssh.braisenly.com` moves to a different local origin.

## Quality Signals
- Useful: one hub plus one entity plus one runbook plus one session note should make future retrieval land on the right document without forcing agents to parse a giant machine profile.
- Useful: root-vs-user `cloudflared` ownership is now explicit, which was the biggest source of confusion during the live repair.
- Gap: the public browser client implementation itself is still implicit; current notes document the transport and host side more strongly than the browser-side app stack.
- Follow-up quality check: watch whether future retrieval queries for `ipad`, `browser terminal`, `ssh.braisenly.com`, or `cloudflared drops` all land on this cluster.

## Curation Primitive Compilation
1. Create Node: created this dedicated hub for the browser-rendered terminal domain.
2. Update Node: related infrastructure notes are updated to point here instead of leaving the knowledge isolated.
3. Relate Nodes: links connect Cloudflare tunnel behavior, SSH behavior, machine profile context, and older Cloudflare smoke evidence.
4. Create Index-Hubs: this note is the canonical entrypoint for the domain.
5. Split/Merge: split the domain into hub + entity + runbook + incident note.
6. Refile/Rename: no refile required; `operations/macos` is the correct operational home.
7. Deprecate/Archive: no deprecations required.
8. Hygiene/Compression: avoided duplicating the full machine profile and SSH manual by linking outward.

## Evolution Log
| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-21 | Created hub for Cloudflare browser-rendered terminal operations on `largo`. | The KB had Cloudflare-adjacent fragments but no canonical note cluster for the live browser-rendering workflow and its stabilization procedures. | User request to build a robust foundational note set around the new `cloudflare browser rendering` entity. |
| 2026-05-28 | Completed frontmatter (added entity_type/created/modified/status); added inbound link to the previously-orphaned `[[Cloudflare Access rollout and rollback protocol for tunnel hostnames]]`; added missing `[[Operational Session - Cloudflare Hostname Alias Cutover on largo (2026-04-29)]]` relation to match the Canonical Notes list | Index was missing two relations and the Access protocol note was an orphan | KB curation hygiene sweep (operations/reference partition) |

## Relations
- related_to [[Cloudflare Browser Rendering on largo]]
- related_to [[Runbook - Cloudflare Browser Rendering Terminal Verification and Recovery on largo]]
- related_to [[Operational Session - Cloudflare Browser Rendering Terminal Stabilization on largo (2026-04-21)]]
- related_to [[Operational Session - Cloudflare Hostname Alias Cutover on largo (2026-04-29)]]
- related_to [[Cloudflare Access rollout and rollback protocol for tunnel hostnames]]
- related_to [[Machine Profile - largo]]
- related_to [[SSH Remote Access Recovery and Operations Manual (Windows/Linux/macOS)]]
- related_to [[Index – Agent KB]]
