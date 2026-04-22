---
title: Python Expert Learning Path — Session Context 2026-04-04
type: session
permalink: kb/sessions/python-expert-learning-path-session-context-2026-04-04
created: '2026-04-04'
modified: '2026-04-04'
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/python
- domain/applied-ai
- domain/education
- status/evergreen
- session/2026-04-04
---

# Python Expert Learning Path — Session Context 2026-04-04

## Session Overview

**Date:** 2026-04-04
**User:** Luke Zautke (me@lukezautke.com, GitHub: zautke)
**Request:** A non-obvious, idiosyncratic learning path for a 13-year senior polyglot engineer (7 languages) who has never touched Python, targeting senior-level Python expertise with applied AI specialization.
**Method:** 3-layer iterative web research (each layer's output fed the next), producing an exhaustive 3,132-line guide across 25 parts + appendices.

## User Context (Critical for Future Agents)

- **Background:** Expert-level. Fluent in concepts: memory models, async, type systems, metaprogramming, concurrency primitives, package management. Does NOT need basics explained.
- **Explicit exclusions:** No beginner content. No "what is a variable." No conventional treatment of conventional topics.
- **Explicit inclusions:** Gotchas, non-intuitive behaviors, CPython internals, exhaustive async control flow, full decorator treatise, `uv` tool comprehensive coverage, all Python idiosyncrasies, applied AI stack.
- **Tool preferences:** `pnpm` for JS, `uv` for Python, `tsup` for TypeScript bundling, `unknown` over `any` in TypeScript, Alpine Docker base images, `docker compose` (not `docker-compose`).
- **Session notes tracking:** `CLAUDE_NOTES.md` convention per user preferences.

## Document Structure Produced

The learning path was delivered in two files:
- `python-expert-learning-path.md` — Parts 1–11 + Appendix A (3,132 lines total including Part 2 content)
- `python-expert-learning-path-part2.md` — Parts 12–25 + Appendix B (standalone)

### Part Coverage Map

| Part | Topic | Key Non-Obvious Content |
|------|-------|------------------------|
| 1 | Python Object Model | Name binding vs assignment, CPython int cache (-5 to 256), string interning rules, mutable defaults, late binding closures, LEGB assignment trap, `__slots__`, metaclass wheel |
| 2 | `uv` Toolchain | PEP 723 inline deps, workspace/monorepo lockfile, `uvx` vs `uv run` distinction, content-addressable cache, `exclude-newer` for reproducibility |
| 3 | Type System | `TypeVar` bounds/variance, `ParamSpec`/`Concatenate` for decorator typing, `Protocol` runtime checks only check presence not signature, `TypeIs` vs `TypeGuard` bidirectionality, `Annotated`, `type` statement (3.12+), `from __future__ import annotations` trap |
| 4 | Generator/Coroutine Protocol | `send`/`throw`/`close`, `yield from` as bidirectional channel (PEP 380), `__await__`, async generator GC leak |
| 5 | Async Control Flow | Event loop architecture, `get_running_loop()` vs `get_event_loop()`, task weak-ref GC bug, `CancelledError` as `BaseException`, `shield()` semantics, `TaskGroup` vs `gather()`, `asyncio.timeout()` vs `wait_for()` hang bug, `contextvars` propagation rules |
| 6 | Error Handling | Exception hierarchy, `__cause__`/`__context__`/`__suppress_context__`, `ExceptionGroup`/`except*` (PEP 654), `add_note()`, exception variable reference cycle deletion, `sys.exception()` (3.11+) |
| 7 | Decorators | Bottom-up application order, `functools.wraps` exact behavior, `ParamSpec`-typed decorator factories, descriptor protocol (data vs non-data priority), `@property` internals, `lru_cache` on instance methods memory leak, `cached_property` thread-safety, class decorators vs metaclass vs `__init_subclass__` |
| 8 | CPython Internals | GIL/PEP 703 status (3.13 experimental, 3.14 improved), JIT (3.13+ ~5%), peephole optimizer/constant folding, import system/`sys.modules` |
| 9 | Standard Library | `collections.ChainMap`, `deque(maxlen=)`, `itertools` lazy pipelines, `pathlib` over `os.path` |
| 10 | Applied AI Stack | SOTA 2026 stack, Pydantic v2 migration gotchas (v1→v2 API changes), async LLM streaming, NumPy views vs copies, fancy indexing always copies |
| 11 | Version Progression | 3.10–3.14 key features per version |
| 12 | Pattern Matching | Mapping patterns are partial, bare names are captures not lookups, OR pattern binding consistency |
| 13 | Attribute Lookup | 6-step chain, `__getattr__` vs `__getattribute__`, `__missing__` |
| 14 | MRO/super() | C3 linearization, super() is "next in MRO" not "parent", `__class__` cell, `__init_subclass__` |
| 15 | Memory Management | RC + cyclic GC, weak references |
| 16 | `contextlib` | `@contextmanager` must yield exactly once, `ExitStack`, `aclosing` for async generators |
| 17 | Advanced Typing | `TypedDict`, `NamedTuple` vs `dataclass` matrix, `Self`, `overload`, `Never`/`assert_never`, `cast` as runtime no-op |
| 18 | `inspect` | `signature`, `unwrap`, predicates for async/generator detection |
| 19 | Performance | cProfile/py-spy hierarchy, local binding optimization, `mypyc` AOT |
| 20 | Concurrency Primitives | asyncio Lock/Event/Condition/Semaphore, threading.Lock in async = event loop block |
| 21 | `anyio` | Library vs application async backend choice |
| 22 | Packaging | `pyproject.toml`, `src` layout, build backends |
| 23 | `dataclasses` | `field()` full options, `frozen` + `__post_init__`, inheritance ordering `TypeError` |
| 24 | Ecosystem | `httpx`, `structlog`, `ruff`, `mypy` vs `pyright` |
| 25 | Applied AI Patterns | Streaming FastAPI endpoint, structured output + retry, `TaskGroup` batch with semaphore, eval-first engineering |

## Key Decisions Made

- **Scope:** Exhaustively covered ALL non-obvious aspects; no basics omitted that would be non-obvious
- **Format:** Code-heavy with inline explanation; no bullet-point summaries of trivial things
- **Research method:** 3-layer iterative web search (6 parallel queries per layer, 18 total)
- **Target file:** Markdown `.md` for maximum portability and rendering in any dev environment

## Observations

- [best-practice] 3-layer iterative research with each layer as a feedback loop for the next produced significantly better signal than single-pass; Layer 1 surfaced keywords (wtfpython, PEP 703, TaskGroup) that Layer 2 deep-dived
- [best-practice] For expert-level content requests, leading with "what your background does NOT prepare you for" frames the content correctly
- [anti-pattern] Treating a polyglot expert's Python request as a "beginner Python" request wastes both tokens and the user's time
- [finding] The `uv` PEP 723 inline script dependencies feature is widely unknown even among experienced Python engineers as of April 2026
- [finding] The TaskGroup vs gather() structured cancellation distinction is one of the most common sources of subtle async bugs in production Python code
- [finding] Python 3.14 is in active development; free-threading overhead improved from ~40% (3.13) to ~5-10% (3.14)

## Relations

- related_to [[Python Applied AI Reference 2026]]
- related_to [[Python Research Sources 2026-04-04]]
- related_to [[t+1 Knowledge Growth — Python Expert Domain 2026-04-04]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------| 
| 2026-04-04 | Initial creation | Session documentation of 3-layer research sprint and 25-part guide production | User request for Python expert learning path with KB notes |
