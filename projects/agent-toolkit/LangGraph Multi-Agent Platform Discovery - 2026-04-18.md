---
title: LangGraph Multi-Agent Platform Discovery - 2026-04-18
type: discovery
permalink: kb/projects/agent-toolkit/lang-graph-multi-agent-platform-discovery-2026-04-18
---

# LangGraph Multi-Agent Platform Discovery - 2026-04-18

Synced from local repo artifact: `/home/luke/dev/langgraph/DISCOVERY.md`
Sync timestamp: 2026-04-18 MDT

---

# Discovery

Date: 2026-04-18

## Scope

Greenfield discovery for a production-grade, TypeScript-first multi-agent orchestration platform with:

- explicit orchestration
- internal and external tool routing
- live dashboarding
- full traceability of agent actions, communications, memory, approvals, and outcomes
- persistent database-backed observability

## Local State

- Repository state: effectively empty.
- `code-graph` quick analysis completed, but there is no local code to analyze yet.
- `jcodemunch` indexing failed with `No source files found`.
- This is a net-new implementation, not an extension of an existing codebase.

## Key External Findings

### 1. LangGraph JS remains the right orchestration substrate

Official LangGraph JS docs still center multi-agent systems on four patterns:

- network
- supervisor
- hierarchical
- custom workflow

The core primitive is `Command`, which couples:

- control flow via `goto`
- state mutation via `update`
- graph boundary crossing via `graph: Command.PARENT`

That makes LangGraph suitable for explicit handoffs, resumability, checkpointing, and human approval flows.

Implication:

- Use low-level `StateGraph` + `Command` for the platform core.
- Use `@langchain/langgraph-supervisor` and `@langchain/langgraph-swarm` only as references or optional starter adapters, not as the architecture boundary.

### 2. The Mintlify custom-agent pattern is configuration-first

The Oh My OpenCode custom-agent design is factory-based:

- `createXAgent(model) -> AgentConfig`
- explicit agent metadata
- per-agent tool restrictions
- model-specific reasoning/thinking settings
- dynamic prompt construction from available agents/tools/skills

Implication:

- Model each agent in your platform as declarative config plus runtime bindings.
- Keep prompts, permissions, tools, memory policy, reasoning budget, and model routing separate from orchestration code.

### 3. SOTA production guidance is not “more agents”

The strongest recurring market signal is that production systems win on:

- tight orchestration
- durable execution
- evals and quality gates
- observability and replay
- protocol boundaries for tools and agent handoffs

The weakest pattern is uncontrolled mesh collaboration. It is harder to debug, harder to secure, and harder to evaluate.

Implication:

- Start with a hub-and-spoke topology.
- Add hierarchy only where there is real decomposition value.
- Do not start with an unrestricted swarm.

## Recommended Architecture

### Topology

Use a hierarchical hub-and-spoke system:

- `Host/Coordinator (The Concierge)` as the primary user-facing orchestrator
- specialist workers for research, coding, architecture, curation
- two distinct quality gates:
  - `The Curmudgeon`: adversarial critic, regression finder, contradiction hunter
  - `The Overseer`: policy/compliance/release gate, approval routing, final risk check

### Runtime layers

1. Control plane

- agent registry
- prompt/config management
- tool registry
- run policies
- human approval controls

2. Orchestration plane

- LangGraph `StateGraph`
- explicit node graph per workflow
- `Command`-based handoffs
- interrupt/resume for approvals and recovery

3. Tool plane

- common internal tools exposed through a typed tool SDK
- MCP adapter layer for external servers
- per-agent tool allowlists and deny lists
- normalized tool-call event schema

4. Memory plane

- short-term thread state
- episodic run memory
- semantic memory
- knowledge graph / entity graph

5. Observability plane

- append-only event log
- OpenTelemetry spans
- run replay
- live websocket event fan-out
- evals and scorecards

## Recommended Persistence Model

Use Postgres as the canonical system of record.

Minimum tables:

- `sessions`
- `runs`
- `run_steps`
- `agents`
- `agent_versions`
- `messages`
- `handoffs`
- `tool_calls`
- `tool_results`
- `interrupts`
- `approvals`
- `memories`
- `memory_links`
- `evals`
- `scores`
- `artifacts`
- `trace_spans`
- `events`
- `config_snapshots`

Recommended supporting stores:

- `Qdrant` for semantic memory and retrieval
- `Neo4j` for entity/relationship memory, task graph lineage, and cross-run knowledge linking
- `Redis` only for ephemeral pub/sub, stream fan-out, and hot caches, not as source of truth

## Observability Requirements

Everything should be represented twice:

- as a domain event in the append-only event log
- as a trace/span/log signal for operational debugging

Capture at minimum:

- user inputs
- system prompt version and resolved agent config
- reasoning mode / budget metadata
- every handoff
- every tool call and raw result
- every memory read/write
- every human interrupt/approval
- every model call with latency/token/cost metadata
- every quality evaluation
- every error, retry, timeout, and cancellation

Do not make “thinking” a hidden blob. Store it as structured internal reasoning artifacts with retention and redaction policy controls.

## Dashboard Surface

The dashboard should not be an afterthought. It is part of the platform.

Required views:

- live run timeline
- agent conversation stream
- handoff graph
- trace/span explorer
- tool execution panel
- memory reads/writes panel
- config diff and agent version viewer
- approvals inbox
- eval scorecards
- replay / deterministic reconstruction view

## Agent Set

### Core orchestrator

- `The Concierge`: user-facing host, routing, clarification, workflow assembly

### Specialist agents

- `The Curator`: retrieval, synthesis, knowledge shaping, source hygiene
- `The Wizard`: advanced research, cross-source investigation, evidence assembly
- `Knuth`: implementation, debugging, code transformation
- `The Architect`: system decomposition, interface design, technical tradeoffs

### Gates

- `The Curmudgeon`: skeptical reviewer, contradiction finder, bug/risk hunter
- `The Overseer`: release/policy/approval gate, governance and operational safety

## Tooling Model

### Common internal tools

Every agent should share a typed internal toolkit:

- run/session lookup
- event/query APIs
- memory search and memory write
- artifact read/write
- trace lookup
- eval submission
- handoff request
- approval request

### External MCP tools

Wrap MCP servers behind an internal adapter so agents do not depend on raw MCP server quirks.

Required adapter behavior:

- capability discovery
- timeout / retry policy
- auth isolation
- structured result normalization
- event logging
- tool-level policy enforcement

### Specialized tools by role

- `The Wizard`: search, crawl, fetch, source comparison, citation assembly
- `Knuth`: repo/code tools, tests, diffs, refactors
- `The Architect`: dependency analysis, design docs, graph/model inspection
- `The Curator`: retrieval, dedupe, summarization, taxonomy tools
- `The Curmudgeon`: evals, invariants, test synthesis, contradiction checks
- `The Overseer`: policy, approval, audit, release gating

## Initial Build Recommendation

Start with this sequence:

1. Define the canonical event schema and Postgres model.
2. Define the agent config schema and versioning model.
3. Implement the common internal tool SDK.
4. Build the LangGraph orchestrator core with explicit `Command` handoffs.
5. Add interrupt/resume and approval flows.
6. Add websocket-backed live run streaming.
7. Add Qdrant and Neo4j memory adapters behind the memory plane.
8. Add eval pipelines and quality gates.
9. Build the dashboard on top of the event log and trace store.

## Current Blockers

- `basic-memory` MCP was not usable from this session; startup/handshake failed during resource discovery.
- Because of that, discovery artifacts were not duplicated into basic memory.
- Do not create `PLANNING.md`, `TASKS.md`, or `SESSION_HISTORY.md` until the plan is accepted.

## Sources

- Mintlify custom agents:
  - https://mintlify.wiki/code-yeongyu/oh-my-opencode/advanced/custom-agents
- LangGraph JS overview:
  - https://docs.langchain.com/oss/javascript/langgraph/overview
- LangGraph JS interrupts:
  - https://docs.langchain.com/oss/javascript/langgraph/interrupts
- LangGraph JS graph API:
  - https://docs.langchain.com/oss/javascript/langgraph/graph-api
- LangGraph JS multi-agent concepts:
  - https://github.com/langchain-ai/langgraphjs/blob/main/docs/docs/concepts/multi_agent.md
- LangGraph JS multi-agent agents docs:
  - https://github.com/langchain-ai/langgraphjs/blob/main/docs/docs/agents/multi-agent.md