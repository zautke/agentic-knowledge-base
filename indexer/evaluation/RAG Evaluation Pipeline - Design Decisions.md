---
type: design-decisions
tags:
- rag
- evaluation
- architecture
- design
- indexer
- decisions
created: 2026-01-21
module: src/evaluation
team: DOC-002
permalink: agent-kb/indexer/evaluation/rag-evaluation-pipeline-design-decisions
---

# RAG Evaluation Pipeline - Design Decisions

This document captures key architectural and design decisions made during the implementation of the RAG Evaluation Pipeline.

## Decision 1: Dual Logging System

### Context
Evaluation results need to be consumed by both humans (debugging, monitoring) and machines (CI/CD, analysis scripts).

### Options Considered
1. **Console only** - Simple but loses structured data
2. **JSON file only** - Machine-readable but hard to monitor
3. **Dual output** - Both console and JSON simultaneously

### Decision
Implemented dual logging with colored console output AND structured JSON file output.

### Rationale
- [design] Console provides real-time feedback with progress bars and colors
- [design] JSON enables programmatic analysis and historical comparison
- [fact] Minimal overhead since both writers share the same data structure

### Consequences
- Slightly more complex logger implementation
- Users get best of both worlds
- CI/CD can parse JSON while developers watch console

---

## Decision 2: JSON File Storage Over Database

### Context
Need persistent storage for evaluation results to enable historical comparison.

### Options Considered
1. **SQLite** - Structured queries, ACID compliance
2. **JSON files** - Simple, portable, human-readable
3. **In-memory only** - Fast but loses data

### Decision
Implemented JSON file-based storage with an index file for fast lookups.

### Rationale
- [design] JSON files are portable and can be version-controlled
- [design] Index file provides O(1) lookup without loading all results
- [insight] SQLite adds complexity for minimal benefit at expected scale

### Consequences
- Easy to inspect and debug results manually
- Can be committed to git for reproducibility
- May need migration to SQLite if scale exceeds thousands of evaluations

---

## Decision 3: Graded Relevance Support for nDCG

### Context
nDCG metric can use binary relevance (relevant/not) or graded relevance (highly relevant, relevant, marginal).

### Options Considered
1. **Binary only** - Simpler, all relevant docs treated equally
2. **Graded only** - More nuanced but requires grade assignments
3. **Both supported** - Flexible but more complex API

### Decision
Support both binary and graded relevance, defaulting to binary if grades not provided.

### Rationale
- [design] Binary is simpler for basic use cases
- [design] Graded relevance enables nuanced quality assessment
- [fact] Implementation cost minimal - just check if grades provided

### Consequences
- Test cases can optionally include `relevanceGrades` map
- nDCG more meaningful when grades reflect actual relevance levels
- Backwards compatible with binary-only test cases

---

## Decision 4: Fixed K-Values for Metrics

### Context
Precision@k and Recall@k need standardized k values for comparison.

### Options Considered
1. **User-specified k** - Flexible but inconsistent across runs
2. **Fixed standard values** - Consistent but less flexible
3. **Both** - Fixed plus custom

### Decision
Use fixed k values: 1, 3, 5, 10, 20 for all metrics.

### Rationale
- [design] Industry standard values enable benchmarking
- [fact] k=10 is most commonly used for comparison
- [insight] Custom k values can be computed manually if needed

### Consequences
- Results are comparable across different evaluation runs
- Slightly less flexible for unusual use cases
- Consistent with academic IR evaluation practices

---

## Decision 5: Streaming Progress Callbacks

### Context
Long-running evaluations need progress visibility without blocking.

### Options Considered
1. **No progress** - Simple but poor UX
2. **Polling** - User must check periodically
3. **Callbacks** - Real-time push updates

### Decision
Implemented callback-based progress updates with `onProgress`, `onTestCaseComplete`, etc.

### Rationale
- [design] Callbacks enable real-time UI updates
- [design] Non-blocking allows parallel processing
- [fact] Progress percentage and ETA are computed automatically

### Consequences
- Consumers can implement custom progress displays
- Logger uses callbacks for progress bar updates
- Optional - consumers can ignore callbacks if not needed

---

## Decision 6: Comparison-First Storage Design

### Context
Primary use case is comparing evaluation runs to detect regressions.

### Options Considered
1. **Store results only** - Query manually for comparison
2. **Built-in comparison** - First-class support for deltas
3. **External analysis** - Export and use separate tools

### Decision
Built comparison directly into storage with `compare()` method and `getTrend()` for historical analysis.

### Rationale
- [design] Comparison is the primary use case
- [insight] Delta calculation catches regressions early
- [fact] Trend analysis shows improvement over time

### Consequences
- Easy regression detection in CI/CD
- Historical trends visible without external tools
- Storage tracks improved/regressed/unchanged test cases

---

## Observations

- [architecture] Module follows single-responsibility: types, metrics, logging, storage, harness
- [pattern] Factory functions preferred over direct class instantiation
- [insight] All metrics normalized to 0-1 range for consistent interpretation
- [requirement] TypeScript strict mode ensures type safety across the module

## Relations

- [[RAG Evaluation Pipeline - Module Overview]] - Module structure overview
- [[RAG Evaluation Metrics Reference]] - Detailed metric algorithms
- [[rag-evaluation-pipeline-api-guide]] - How to use the API