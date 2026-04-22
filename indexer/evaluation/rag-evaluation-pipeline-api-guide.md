---
type: api-reference
tags: [rag, evaluation, api, typescript, indexer, metrics, benchmark]
created: 2026-01-21
module: src/evaluation
permalink: memory://indexer/evaluation/rag-evaluation-pipeline-api-guide
---

# RAG Evaluation Pipeline - API Guide

Comprehensive API reference for the RAG Evaluation Pipeline module located at `src/evaluation/`. This module provides tools for evaluating retrieval-augmented generation systems with standard IR metrics, dual logging, and persistent storage.

## Quick Start

```typescript
import {
  createEvaluationHarness,
  createEvaluationStorage,
  createTestCasesFromMap,
  EvaluationLogger,
} from './evaluation';

// Create test cases from query-relevance mapping
const testCases = createTestCasesFromMap({
  'authentication logic': ['chunk-auth-1', 'chunk-auth-2'],
  'database connection': ['chunk-db-1'],
});

// Create harness with storage and logging
const harness = createEvaluationHarness({
  retriever,
  storage: createEvaluationStorage({
    type: 'json-file',
    basePath: './eval-results',
  }),
  logger: EvaluationLogger.createDefault({
    jsonFilePath: './eval-output.json',
  }),
});

// Run evaluation
const result = await harness.runRetrievalEvaluation({
  testCases,
  config: { retrieval: { topK: 10, mode: 'hybrid' } },
});

console.log(`Pass rate: ${result.summary.passRate}`);
console.log(`MRR: ${result.retrievalMetrics?.mrr}`);
```

## Core Classes

### EvaluationHarness

Main orchestrator for running RAG evaluation pipelines. Coordinates test case execution, metrics computation, and result logging.

**Constructor:**
```typescript
new EvaluationHarness(config: HarnessConfig)
```

**HarnessConfig:**
| Property | Type | Description |
|----------|------|-------------|
| `retriever` | `Retriever` | Retriever instance to evaluate (required) |
| `storage` | `EvaluationStorage` | Optional storage for persisting results |
| `logger` | `EvaluationLogger` | Optional logger for output |
| `callbacks` | `EvaluationCallbacks` | Event callbacks |
| `defaultTimeoutMs` | `number` | Default timeout per test case (default: 30000) |

**Key Methods:**

- `runRetrievalEvaluation(options)` - Run complete retrieval evaluation
- `compareWithBaseline(result, baselineId?)` - Compare against a baseline
- `getMetricTrend(metric, limit)` - Get historical trend for a metric

### MetricsCalculator

Computes standard information retrieval metrics per-query and aggregates across test suites.

**Methods:**

- `computeMetrics(retrieved, testCase)` - Compute all metrics for a single query
- `aggregateMetrics(results)` - Aggregate metrics across multiple test case results

**Computed Metrics:**
- Precision@k (k=1, 3, 5, 10, 20)
- Recall@k (k=1, 3, 5, 10, 20)
- MRR (Mean Reciprocal Rank)
- nDCG@k (k=5, 10, 20)
- Hit Rate
- Average Precision

### EvaluationLogger

Dual logging system providing simultaneous console and JSON file output.

**Static Factory:**
```typescript
EvaluationLogger.createDefault(options?: Partial<LoggerConfig>)
```

**LoggerConfig:**
| Property | Type | Description |
|----------|------|-------------|
| `console` | `boolean` | Enable console output |
| `consoleLevel` | `ConsoleLevel` | 'minimal' \| 'normal' \| 'verbose' |
| `colors` | `boolean` | Enable ANSI colors (default: true) |
| `jsonFile` | `boolean` | Enable JSON file output |
| `jsonFilePath` | `string` | Path for JSON output file |
| `includeTestCaseResults` | `boolean` | Include individual results in JSON |
| `timestampFormat` | `string` | 'iso' \| 'time' \| 'none' |

### JsonFileStorage

JSON file-based storage for evaluation results. Implements `EvaluationStorage` interface.

**Interface Methods:**
- `save(evaluation, metadata?)` - Save an evaluation result
- `get(id)` - Retrieve evaluation by ID
- `query(query)` - Query evaluations with filters
- `delete(id)` - Delete an evaluation
- `getLatest(type?)` - Get latest evaluation
- `compare(baselineId, comparisonId)` - Compare two evaluations
- `getTrend(metric, limit)` - Get historical trend
- `saveSuite(suite)` / `getSuite(id)` / `listSuites()` - Benchmark suite management

## Factory Functions

### createEvaluationHarness(config)

Create a new evaluation harness instance.

```typescript
const harness = createEvaluationHarness({
  retriever,
  storage,
  logger,
  callbacks: {
    onProgress: (progress) => { /* ... */ },
    onComplete: (result) => { /* ... */ },
  },
});
```

### createTestCasesFromMap(queries)

Create test cases from a simple query-to-relevant-docs mapping.

```typescript
const testCases = createTestCasesFromMap({
  'how does user authentication work': [
    'chunk-auth-login-handler',
    'chunk-auth-middleware',
  ],
  'database connection pooling': [
    'chunk-db-pool-config',
  ],
});
```

### runQuickEvaluation(retriever, testCases, outputPath?)

Run a quick evaluation with minimal configuration.

```typescript
const result = await runQuickEvaluation(
  retriever,
  testCases,
  './quick-eval.json'
);
```

### createEvaluationStorage(config)

Create storage instance.

```typescript
const storage = createEvaluationStorage({
  type: 'json-file',
  basePath: './benchmark-results',
  maxEvaluations: 100,
});
```

### createMetricsCalculator()

Create a metrics calculator instance.

```typescript
const calc = createMetricsCalculator();
const metrics = calc.computeMetrics(retrievedIds, testCase);
```

### createEvaluationLogger(config)

Create a logger with custom configuration.

```typescript
const logger = createEvaluationLogger({
  console: true,
  consoleLevel: 'verbose',
  jsonFile: true,
  jsonFilePath: './eval.json',
});
```

## Metric Functions

Standalone functions for computing individual metrics:

| Function | Description |
|----------|-------------|
| `precisionAtK(retrieved, relevant, k)` | Precision at position k |
| `recallAtK(retrieved, relevant, k)` | Recall at position k |
| `reciprocalRank(retrieved, relevant)` | Reciprocal rank of first relevant doc |
| `dcgAtK(retrieved, grades, k)` | Discounted Cumulative Gain |
| `idealDcgAtK(grades, k)` | Ideal DCG |
| `ndcgAtK(retrieved, grades, k)` | Normalized DCG |
| `averagePrecision(retrieved, relevant)` | Average Precision |
| `hitRate(retrieved, relevant)` | Whether any relevant doc was found |
| `calculateLatencyMetrics(latencies)` | Compute latency statistics |

## Formatting Functions

```typescript
import { formatMetrics, formatAggregatedMetrics } from './evaluation';

// Format single query metrics
console.log(formatMetrics(result.metrics));

// Format aggregated metrics table
console.log(formatAggregatedMetrics(result.retrievalMetrics));
```

## Storage Utilities

```typescript
import {
  exportEvaluation,
  importEvaluation,
  formatComparisonReport,
} from './evaluation';

// Export to standalone file
exportEvaluation(storedEval, './export.json');

// Import from file
const imported = importEvaluation('./export.json');

// Format comparison report
const comparison = await storage.compare(baselineId, currentId);
console.log(formatComparisonReport(comparison));
```

## Complete Example: Running a Benchmark

```typescript
import {
  createEvaluationHarness,
  createEvaluationStorage,
  EvaluationLogger,
  createTestCasesFromMap,
  formatComparisonReport,
  type RetrievalTestCase,
} from './evaluation';

async function runBenchmark(retriever: Retriever) {
  // Define test cases with graded relevance
  const testCases: RetrievalTestCase[] = [
    {
      id: 'tc-001',
      query: 'implement user session management',
      relevantDocIds: [
        'chunk-session-manager',
        'chunk-session-store',
      ],
      relevanceGrades: {
        'chunk-session-manager': 3,  // highly relevant
        'chunk-session-store': 2,    // relevant
      },
      category: 'sessions',
      tags: ['graded', 'sessions'],
    },
  ];

  // Initialize storage and logger
  const storage = createEvaluationStorage({
    type: 'json-file',
    basePath: './benchmark-results',
  });

  const logger = EvaluationLogger.createDefault({
    jsonFilePath: './benchmark-results/latest.json',
    consoleLevel: 'normal',
  });

  // Create harness
  const harness = createEvaluationHarness({
    retriever,
    storage,
    logger,
    callbacks: {
      onComplete: (result) => console.log('Evaluation complete!'),
    },
  });

  // Run evaluation with thresholds
  const result = await harness.runRetrievalEvaluation({
    testCases,
    iterations: 1,
    config: {
      retrieval: {
        topK: 10,
        mode: 'hybrid',
        hybridAlpha: 0.5,
      },
    },
    thresholds: {
      minMrr: 0.3,
      minPrecision: 0.2,
      minRecall: 0.1,
      maxLatencyMs: 5000,
    },
    gitInfo: {
      sha: process.env.GIT_SHA,
      branch: process.env.GIT_BRANCH,
    },
  });

  // Compare with previous run
  const comparison = await harness.compareWithBaseline(result);
  if (comparison) {
    console.log(formatComparisonReport(comparison));
  }

  // View metric trends
  const trend = await harness.getMetricTrend('mrr', 5);
  trend.forEach((point) => {
    console.log(`${point.date.toLocaleDateString()}: ${point.value.toFixed(4)}`);
  });

  return result;
}
```

## Type Exports

The module exports all relevant types:

- **Core:** `EvaluationId`, `TestCaseId`, `EvaluationType`, `EvaluationStatus`
- **Test Cases:** `RetrievalTestCase`, `IngestionTestCase`, `BenchmarkSuite`
- **Metrics:** `RetrievalMetrics`, `AggregatedMetrics`, `LatencyMetrics`, `PerformanceMetrics`
- **Results:** `TestCaseResult`, `EvaluationResult`, `EvaluationConfig`, `EvaluationProgress`
- **Storage:** `StoredEvaluation`, `EvaluationQuery`, `EvaluationComparison`
- **Config:** `HarnessConfig`, `RetrievalEvaluationOptions`, `LoggerConfig`, `StorageConfig`

## Relations

- [[rag-evaluation-pipeline-overview]] - Module architecture overview
- [[rag-evaluation-metrics-reference]] - Detailed metrics documentation
- [[indexer-retrieval-module]] - Retriever interface used by harness
