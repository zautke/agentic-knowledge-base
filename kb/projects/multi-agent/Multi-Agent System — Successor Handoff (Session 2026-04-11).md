---
created: 2026-04-11
modified: 2026-05-28
entity_type: handoff
status: active
title: Multi-Agent System — Successor Handoff (Session 2026-04-11)
type: handoff
permalink: kb/projects/multi-agent/successor-handoff-2026-04-11
tags:
- handoff
- multi-agent
- langgraph
- python
- websocket
- nemotron
- demo-blocked
- source/largo
- machine/largo
---

# Multi-Agent System — Successor Handoff

## Where Everything Is

```
C:\Users\me\dev\python\
├── backend/                    ← FastAPI + LangGraph
│   ├── src/multi_agent/        ← all source (src-layout)
│   ├── .venv/                  ← project virtualenv (USE THIS, not conda ai env)
│   ├── .env                    ← local dev config (Ollama relay, local Postgres)
│   └── pyproject.toml
├── frontend/                   ← React + Vite (assumed)
├── CLAUDE_NOTES.md             ← session decisions log (read this first)
├── AGENT_TASK_LIST.md          ← prioritized task backlog with acceptance criteria
└── README.md                   ← architecture + Mermaid diagrams
```

## The One Thing That Was Never Done

A screenshot of the chat UI showing a complete agent response with metrics. That is the only outstanding deliverable from two full sessions. Everything else was scaffolding for it.

---

## How To Start The Stack (Exact Commands)

### Backend (this is the command that actually works)
```bash
cd C:\Users\me\dev\python\backend
PYTHONPATH=src .venv/Scripts/uvicorn.exe multi_agent.main:app --host 0.0.0.0 --port 8000
```
- Do NOT use `conda run -n ai` — it breaks on Windows with multiline args
- Do NOT use `--reload` for demos — watchfiles sometimes misses changes on Windows
- Redirect to a log file if you need visibility: append `> C:\tmp\uvi.log 2>&1 &`
- The `.venv` inside `backend/` has all packages. The conda `ai` env does NOT.

### Postgres (local demo, port 5433)
```bash
docker run -d -p 5433:5432 \
  -e POSTGRES_USER=multi_agent \
  -e POSTGRES_PASSWORD=multi_agent \
  -e POSTGRES_DB=multi_agent \
  adagio:5443/braisenly/pg:latest
```

### Ollama cloud relay
Already running on the host machine at port 11435. Do not restart it.
Available models (confirmed working on this machine):
- `nemotron-3-nano:30b-cloud` — current default, has loop issues (see below)
- `qwen3-next:80b-cloud` — best tool-calling behavior, 2 LLM calls per request
- `deepseek-v3.1:671b-cloud` — slow, generates tool calls as prose (broken for agents)
- `gpt-oss:120b-cloud` — untested
- `ministral-3:14b-cloud` — untested

### Frontend
```bash
cd C:\Users\me\dev\python
pnpm dev    # or npm run dev
```
Runs at http://localhost:3000. The Claude Preview server ID is `2a9b1414-b954-4398-bf86-d131a56f05b6` (may have changed after restart).

---

## Current Model Setting

`backend/.env` line 6:
```
MA_LLM_MODEL=ollama/nemotron-3-nano:30b-cloud
```

**The user requires Nemotron.** If you need to unblock the demo quickly, temporarily switch to `ollama/qwen3-next:80b-cloud` — it works cleanly — then switch back. Do not leave it on Qwen3.

---

## What Works

- Backend starts clean. Health check at `http://localhost:8000/health` returns `{"status":"ok"}`.
- All 40 tests pass: `cd backend && PYTHONPATH=src .venv/Scripts/pytest`
- WebSocket connects immediately on page load.
- Supervisor correctly routes to agents — confirmed via logs (`transfer_to_knuth`, `transfer_to_scribe`, etc.).
- The `checkpoint_ns` discriminator correctly identifies which agent is which.
- Supervisor content is filtered from the WebSocket stream (both `on_chat_model_stream` and `on_chat_model_end`).
- Batch-mode LLM fallback (`on_chat_model_end`) emits the agent's full response when streaming doesn't fire.
- Frontend renders agent response bubbles, topology diagram, agent toggles.

## What Is Broken / Unverified

### Nemotron recursion loop (P0)
Nemotron makes 40+ LLM calls for simple messages, hitting the 50-step recursion limit. Root causes:

1. **`include_agent_name="inline"` in `graph.py` line 149** — This wraps every agent message in XML: `<name>knuth</name><content>actual text</content>`. When Nemotron sees this XML in its context, it does not recognise it as a terminal response and keeps re-routing. **This line has been removed in the last edit.** Verify it is absent before testing.

2. **Supervisor prompt was too verbose** — "Always explain which agent you are delegating to and why" caused Nemotron to write prose before calling a tool. The prompt has been rewritten to directive-only. Verify `graph.py` supervisor_prompt says "ONLY call one tool, no text" style instructions.

3. `output_mode` changed from `"full_history"` to `"last_message"` in the `create_supervisor()` call. This reduces context size per step and eliminates the feedback loop of full message chains.

**These three changes together should break the loop.** They were applied but the backend was NOT restarted and tested after all three were in place simultaneously. That is the first thing to verify.

### Demo screenshot
Send this message after confirming the backend starts clean:
```
What is the Single Responsibility Principle?
```
Expected flow: Supervisor → transfer_to_knuth → Knuth answers → done.
With Qwen3 this takes ~25 seconds total. With Nemotron unknown (loop may still occur).

### Supervisor XML leaking into chat
Fixed in `websocket.py` via `checkpoint_ns`-based agent name extraction. But this was confirmed working with Qwen3, not yet confirmed with Nemotron after the `include_agent_name` removal. If XML still appears, the filter is working but `include_agent_name` is still set somewhere.

---

## The Key Architecture Discovery (Do Not Lose This)

### `langgraph_node` is always `"agent"` — use `checkpoint_ns` instead

Every single LLM call in the entire graph — supervisor, Knuth, Scribe, all of them — reports `metadata["langgraph_node"] = "agent"`. This is because `create_react_agent` uses "agent" as its internal node name, and all agents are built with `create_react_agent`.

The correct discriminator is `metadata["checkpoint_ns"]`, which contains `"{subgraph_name}:{uuid}"`.

```python
checkpoint_ns = metadata.get("checkpoint_ns", "")
agent_name = checkpoint_ns.split(":")[0] if checkpoint_ns else metadata.get("langgraph_node", "")
```

Confirmed via debug log:
```
supervisor LLM → cns='supervisor:ecdcb317-...' → agent_name = "supervisor"
knuth LLM      → cns='knuth:2bbf5b76-...'      → agent_name = "knuth"
```

This is in `websocket.py` in both `_handle_message` and `_handle_resume`.

### `AgentState` must be `TypedDict`, NOT `BaseModel`

`langgraph-supervisor` auto-generates handoff tools with a `state: dict` Pydantic schema. If `AgentState` is a `BaseModel`, it fails `dict_type` validation on every handoff → infinite supervisor loop. `TypedDict` IS a dict at runtime → passes transparently.

File: `backend/src/multi_agent/state.py`. Do not revert this.

### Nemotron streams tool-call function names as text content

When Nemotron calls a tool, it streams the function name (`analyze_complexity`, `transfer_to_knuth`) as `chunk.content` with empty `tool_call_chunks`. This is non-standard — OpenAI puts these in `tool_call_chunks` with empty content.

The filter in `websocket.py`:
```python
_FUNC_NAME_RE = re.compile(r"^[a-z][a-z0-9_]*$")
is_func_name = bool(chunk_content and _FUNC_NAME_RE.match(chunk_content.strip()))
```
This catches snake_case function names. **Risk:** may drop single-word tokens from true streaming responses. Minimum length guard may be needed if real streaming models are used.

---

## Files Modified in Session 2 (and Why)

| File | What Changed | Why |
|------|-------------|-----|
| `backend/src/multi_agent/state.py` | `BaseModel` → `TypedDict` | Handoff tool validation crash |
| `backend/src/multi_agent/graph.py` | Removed `include_agent_name="inline"`; tightened supervisor prompt; `output_mode` → `"last_message"` | Nemotron XML leak + loop |
| `backend/src/multi_agent/transport/websocket.py` | `checkpoint_ns` agent discrimination; supervisor filter; `_FUNC_NAME_RE`; `nodes_streamed`; `on_chat_model_end` batch fallback | Streaming correctness for batch/hybrid models |
| `backend/src/multi_agent/observability/litellm_setup.py` | Created | Centralise provider credential injection |
| `backend/src/multi_agent/config.py` | Added `gemini_api_key`, `ollama_base_url`, `github_token` | Multi-provider LLM support |
| `backend/src/multi_agent/main.py` | Added `init_litellm(settings)` in lifespan | Provider creds must be set before graph builds |
| `backend/tests/test_tribunal.py` | Replaced `asyncio.coroutine` mocks | Removed in Python 3.11; running 3.14 |
| `backend/tests/test_api.py` | Rewrote client fixture to mock lifespan | Prevented Postgres connection during tests |
| `backend/pyproject.toml` | `pythonpath = ["src"]`; `readme = "../README.md"` | Module discovery + hatchling build |
| `backend/.env` | Created | Local dev config (Ollama relay, local Postgres) |
| `README.md` | Created | Mermaid architecture + sequence + A2A diagrams |

---

## Remaining Task Backlog

See `AGENT_TASK_LIST.md` at the project root for the full prioritised list with acceptance criteria. The P0 items are:

1. **T-001** Remove debug log from websocket.py — DONE (already removed)
2. **T-002** Confirm Nemotron loop is fixed by the three changes above
3. **T-003** Confirm supervisor XML no longer appears in chat UI
4. **T-004** Take the screenshot

P1 items that matter before production:
- **T-005** Fix `_handle_resume` — it still uses old streaming logic without any of the fixes
- **T-006** Wire Postgres checkpointer (currently `InMemorySaver` — state lost on restart)
- **T-008** Re-run test suite after Session 2 changes

---

## What To Do First (Literally The Next 15 Minutes)

```
1. Confirm graph.py has NO include_agent_name="inline" on line ~149
2. Confirm supervisor_prompt is directive-only (no "explain your reasoning")
3. Start backend: PYTHONPATH=src .venv/Scripts/uvicorn.exe multi_agent.main:app --host 0.0.0.0 --port 8000 > C:\tmp\uvi.log 2>&1
4. grep "Active LLM" C:\tmp\uvi.log  ← confirm nemotron-3-nano:30b-cloud
5. Open http://localhost:3000
6. Send: "What is the Single Responsibility Principle?"
7. Watch C:\tmp\uvi.log — count how many "LiteLLM completion()" lines appear
8. If <= 4 lines: loop is fixed. Wait for response, take screenshot.
9. If > 10 lines: loop persists. Switch model to qwen3-next:80b-cloud in .env, restart, screenshot, switch back.
```

---

## Relations
- documented_by [[CLAUDE_NOTES.md]]
- task_backlog [[AGENT_TASK_LIST.md]]
- failure_pattern [[Mistaking Motion for Progress]]