---
title: RAG Evaluation Pipeline - Module Overview
type: technical-documentation
tags:
- rag
- evaluation
- metrics
- retrieval
- indexer
- code-search
- information-retrieval
created: 2026-01-21
module: src/evaluation
team: DOC-002
permalink: agent-kb/indexer/evaluation/rag-evaluation-pipeline-module-overview
---

# RAG Evaluation Pipeline - Module Overview

## Overview

The RAG Evaluation Pipeline is a comprehensive evaluation system for retrieval-augmented generation (RAG) systems within the indexer project. Located at `src/evaluation/`, this module provides standardized information retrieval metrics, dual logging capabilities, persistent storage for benchmark results, and historical trend analysis.

The module is designed to measure and track retrieval quality over time, enabling data-driven improvements to code search and RAG pipelines. It implements industry-standard IR metrics and supports both automated benchmarking and exploratory evaluation workflows.

## Key Features

- **Standard IR Metrics**: Precision@k, Recall@k, MRR (Mean Reciprocal Rank), nDCG (Normalized Discounted Cumulative Gain), Average Precision, and Hit Rate
- **Dual Logging System**: Simultaneous console output (human-readable with ANSI colors) and JSON file logging (for programmatic analysis)
- **Persistent Storage**: JSON file-based storage with support for SQLite backends, enabling historical tracking
- **Historical Trend Analysis**: Track metric changes across evaluation runs to identify regressions or improvements
- **Evaluation Comparison**: Compare two evaluation runs with delta calculations and statistical significance testing
- **Flexible Test Cases**: Support for binary relevance and graded relevance (for nDCG calculations)
- **Performance Metrics**: Query latency statistics (avg, median, p95, p99), throughput measurements, and resource usage tracking
- **Progress Callbacks**: Event-driven architecture with callbacks for monitoring long-running evaluations

## Module Structure

The evaluation module consists of six core files:

### types.ts
Comprehensive TypeScript type definitions including:
- Core identifiers: `EvaluationId`, `TestCaseId`, `EvaluationType`, `EvaluationStatus`
- Test case interfaces: `RetrievalTestCase`, `IngestionTestCase`, `BenchmarkSuite`
- Metrics interfaces: `RetrievalMetrics`, `AggregatedMetrics`, `LatencyMetrics`, `PerformanceMetrics`
- Result types: `TestCaseResult`, `EvaluationResult`, `EvaluationConfig`
- Storage types: `StoredEvaluation`, `EvaluationQuery`, `EvaluationComparison`

### metrics.ts
Implements all information retrieval metric calculations:
- `precisionAtK()` - Fraction of top-k results that are relevant
- `recallAtK()` - Fraction of relevant docs in top-k
- `reciprocalRank()` - MRR for single query (1/rank of first relevant result)
- `dcgAtK()`, `idealDcgAtK()`, `ndcgAtK()` - DCG-based ranking quality metrics
- `averagePrecision()` - Area under precision-recall curve
- `hitRate()` - Binary success indicator
- `MetricsCalculator` class for computing and aggregating metrics
- `calculateLatencyMetrics()` - Statistical latency analysis (avg, median, percentiles)

### logger.ts
Dual logging system with:
- Console output with ANSI color formatting and progress bars
- JSON file output for structured data storage
- Three verbosity levels: minimal, normal, verbose
- Real-time progress updates during evaluation
- Timestamp formatting options

### storage.ts
Persistent storage system supporting:
- JSON file-based storage (default)
- SQLite storage backend (optional)
- CRUD operations for evaluation results
- Query interface with filtering by type, status, date range, and tags
- Comparison functionality between evaluation runs
- Trend analysis for tracking metrics over time
- Export/import functionality for portability

### harness.ts
Main evaluation orchestrator providing:
- `EvaluationHarness` class - coordinates test execution, metrics, and logging
- `runRetrievalEvaluation()` - executes full retrieval evaluation pipeline
- Test case filtering by tags
- Configurable thresholds for pass/fail determination
- Environment information capture (Node version, platform, memory, CPU)
- Git integration for tracking commits/branches with results

### index.ts
Public API surface with organized exports:
- Type exports for external consumption
- Class exports: `EvaluationHarness`, `MetricsCalculator`, `EvaluationLogger`, `JsonFileStorage`
- Factory functions: `createEvaluationHarness()`, `createMetricsCalculator()`, `createEvaluationLogger()`, `createEvaluationStorage()`
- Utility functions: `runQuickEvaluation()`, `createTestCasesFromMap()`, metric formatting helpers

## Getting Started

### Basic Usage

```typescript
import {
  createEvaluationHarness,
  createEvaluationStorage,
  EvaluationLogger,
  createTestCasesFromMap,
} from './evaluation';

// Create test cases from query-to-relevant-docs mapping
const testCases = createTestCasesFromMap({
  'authentication logic': ['chunk-auth-1', 'chunk-auth-2'],
  'database connection': ['chunk-db-1'],
  'error handling patterns': ['chunk-error-1', 'chunk-error-2', 'chunk-error-3'],
});

// Create evaluation harness with storage and logging
const harness = createEvaluationHarness({
  retriever,  // Your retriever implementation
  storage: createEvaluationStorage({
    type: 'json-file',
    basePath: './eval-results',
  }),
  logger: EvaluationLogger.createDefault({
    jsonFilePath: './eval-output.json',
    consoleLevel: 'normal',
  }),
});

// Run evaluation
const result = await harness.runRetrievalEvaluation({
  testCases,
  config: {
    retrieval: { topK: 10, mode: 'hybrid', hybridAlpha: 0.5 },
  },
  thresholds: {
    minMrr: 0.3,
    minPrecision: 0.2,
    maxLatencyMs: 5000,
  },
});

// Access results
console.log(`Pass rate: ${(result.summary.passRate * 100).toFixed(1)}%`);
console.log(`MRR: ${result.retrievalMetrics?.mrr.toFixed(4)}`);
console.log(`Precision@10: ${result.retrievalMetrics?.precision['@10'].toFixed(4)}`);
```

### Quick Evaluation

For rapid testing without storage configuration:

```typescript
import { runQuickEvaluation, createTestCasesFromMap } from './evaluation';

const testCases = createTestCasesFromMap({
  'find user model': ['user-model-chunk'],
  'api routes': ['routes-chunk-1', 'routes-chunk-2'],
});

const result = await runQuickEvaluation(retriever, testCases, './quick-eval.json');
```

### Understanding Metrics

| Metric | Range | Description |
|--------|-------|-------------|
| Precision@k | 0-1 | What fraction of top-k results are relevant |
| Recall@k | 0-1 | What fraction of all relevant docs are in top-k |
| MRR | 0-1 | 1 / rank of first relevant result |
| nDCG@k | 0-1 | Position-weighted relevance (favors relevant docs ranked higher) |
| Hit Rate | 0-1 | Did we find at least one relevant result |
| Avg Precision | 0-1 | Area under the precision-recall curve |

## Observations

- [design] The module follows a factory function pattern for object creation, enabling dependency injection and testability
- [design] Dual logging (console + JSON) separates human-readable output from machine-parseable data
- [fact] Standard k values for precision/recall are 1, 3, 5, 10, 20; for nDCG are 5, 10, 20
- [fact] Pass/fail thresholds are configurable per-evaluation with defaults of MRR >= 0.3, Precision >= 0.2, Latency <= 5000ms
- [requirement] Test cases require a query string and list of relevant document IDs; graded relevance is optional
- [insight] The comparison feature enables A/B testing of retrieval configurations by tracking improved/regressed/unchanged test cases

## Relations

- [[RAG Evaluation Metrics Reference]] - Detailed metrics documentation and formulas
- [[rag-evaluation-pipeline-api-guide]] - Complete API usage examples and patterns
- [[RAG Evaluation Pipeline - Design Decisions]] - Architecture decisions and rationale
