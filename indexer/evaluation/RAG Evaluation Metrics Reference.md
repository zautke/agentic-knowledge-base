---
type: algorithm-reference
tags:
- rag
- evaluation
- metrics
- precision
- recall
- mrr
- ndcg
- information-retrieval
- source/largo
- machine/largo
created: 2026-01-21
module: src/evaluation/metrics.ts
permalink: agent-kb/indexer/evaluation/rag-evaluation-metrics-reference
---

# RAG Evaluation Metrics Reference

## Overview

This document explains the retrieval quality metrics implemented in the indexer evaluation pipeline. These metrics are standard information retrieval measures used to assess how well a RAG (Retrieval-Augmented Generation) system retrieves relevant documents for a given query.

All metrics are computed per-query and can be aggregated across a test suite using the `MetricsCalculator` class. The implementation supports both binary relevance (relevant/not relevant) and graded relevance (multiple levels of relevance).

## Precision@k

### Formula

```
Precision@k = |Retrieved@k intersection Relevant| / k
```

### Explanation

Precision@k measures what fraction of the top-k retrieved documents are actually relevant. It answers the question: "Of the documents I retrieved, how many were useful?"

A Precision@5 of 0.6 means that 3 out of the top 5 retrieved documents were relevant.

### When to Use

- When you care about the quality of results shown to users
- When users typically only look at a small number of results
- When false positives (irrelevant results) are costly
- Good for evaluating search result pages where screen space is limited

### Example

Given:
- Retrieved documents (in order): [A, B, C, D, E]
- Relevant documents: {A, C}

Calculation:
- Precision@1 = 1/1 = 1.0 (A is relevant)
- Precision@3 = 2/3 = 0.67 (A, C are relevant out of A, B, C)
- Precision@5 = 2/5 = 0.4 (A, C are relevant out of all 5)

### Implementation Notes

Standard k values used: 1, 3, 5, 10, 20

## Recall@k

### Formula

```
Recall@k = |Retrieved@k intersection Relevant| / |Relevant|
```

### Explanation

Recall@k measures what fraction of all relevant documents appear in the top-k results. It answers the question: "Of all the documents that should have been found, how many did I actually find?"

A Recall@10 of 0.8 means that 80% of all relevant documents appear in the top 10 results.

### When to Use

- When missing relevant documents is costly (e.g., legal discovery, medical research)
- When users need comprehensive coverage of a topic
- When the total number of relevant documents is known
- Often used alongside precision to understand the precision-recall tradeoff

### Example

Given:
- Retrieved documents (in order): [A, B, C, D, E]
- Relevant documents: {A, C, F, G} (4 total relevant docs)

Calculation:
- Recall@1 = 1/4 = 0.25 (found A out of 4 relevant)
- Recall@3 = 2/4 = 0.5 (found A, C out of 4 relevant)
- Recall@5 = 2/4 = 0.5 (still only found A, C; F and G not retrieved)

### Implementation Notes

- If there are no relevant documents, recall is defined as 1.0 (vacuously true)
- Standard k values used: 1, 3, 5, 10, 20

## Mean Reciprocal Rank (MRR)

### Formula

```
MRR = 1 / rank_of_first_relevant_document
```

For multiple queries:
```
MRR = (1/|Q|) * sum over q in Q of [1 / rank_q]
```

### Explanation

MRR measures how high the first relevant result appears in the ranked list. It rewards systems that place at least one relevant document near the top.

An MRR of 0.5 means the first relevant document appears, on average, at position 2.

### When to Use

- When users typically only need one good answer (e.g., fact-finding queries)
- When the position of the first relevant result matters most
- Good for navigational queries where there is one "right" answer
- Useful for question-answering systems

### Example

Given:
- Retrieved documents: [B, A, C, D, E]
- Relevant documents: {A, C}

Calculation:
- First relevant document (A) is at rank 2
- MRR = 1/2 = 0.5

Another example:
- Retrieved: [A, B, C, D, E], Relevant: {A, C}
- First relevant (A) is at rank 1
- MRR = 1/1 = 1.0

### Implementation Notes

- Returns 0 if no relevant document is found in the retrieved list
- Rank is 1-indexed (first position is rank 1)

## Normalized Discounted Cumulative Gain (nDCG)

### Formula

**Discounted Cumulative Gain (DCG):**
```
DCG@k = sum from i=1 to k of [rel_i / log2(i + 1)]
```

**Ideal DCG (IDCG):**
```
IDCG@k = DCG of the ideal ranking (documents sorted by relevance)
```

**Normalized DCG:**
```
nDCG@k = DCG@k / IDCG@k
```

### Explanation

nDCG is a position-weighted metric that accounts for graded relevance. It gives more credit to relevant documents that appear higher in the ranking and can distinguish between highly relevant and somewhat relevant documents.

The "discounting" means that relevance scores are divided by a logarithmic factor based on position, so documents lower in the ranking contribute less to the score.

An nDCG of 1.0 means the ranking is perfect (matches the ideal ordering by relevance).

### When to Use

- When you have graded relevance judgments (e.g., 0-3 scale)
- When the position of all relevant documents matters, not just the first
- When highly relevant documents should be ranked above moderately relevant ones
- Standard metric for web search and recommendation systems

### Example

Given:
- Retrieved documents: [A, B, C, D]
- Relevance grades: A=3, B=0, C=2, D=1

DCG@4 calculation:
```
DCG = 3/log2(2) + 0/log2(3) + 2/log2(4) + 1/log2(5)
    = 3/1 + 0/1.58 + 2/2 + 1/2.32
    = 3 + 0 + 1 + 0.43
    = 4.43
```

Ideal ranking: [A, C, D, B] (sorted by relevance: 3, 2, 1, 0)
```
IDCG = 3/log2(2) + 2/log2(3) + 1/log2(4) + 0/log2(5)
     = 3 + 1.26 + 0.5 + 0
     = 4.76
```

nDCG@4 = 4.43 / 4.76 = 0.93

### Implementation Notes

- Standard k values used: 5, 10, 20
- If no relevance grades are provided, binary relevance (1 for relevant, 0 for not) is used
- Returns 0 if IDCG is 0 (no relevant documents)

## Average Precision (AP)

### Formula

```
AP = (1 / |Relevant|) * sum from k=1 to n of [Precision@k * rel(k)]
```

Where rel(k) = 1 if the document at rank k is relevant, 0 otherwise.

### Explanation

Average Precision computes precision at each position where a relevant document is found, then averages these values. It rewards systems that cluster relevant documents at the top of the ranking.

AP can be thought of as the area under the precision-recall curve.

### When to Use

- When you want a single number that captures both precision and recall
- When the ordering of all relevant documents matters
- Standard metric for document retrieval benchmarks (e.g., TREC)
- When aggregated across queries, becomes Mean Average Precision (MAP)

### Example

Given:
- Retrieved documents: [A, B, C, D, E]
- Relevant documents: {A, C, E} (3 relevant)

Calculation:
- At rank 1: A is relevant, Precision@1 = 1/1 = 1.0
- At rank 2: B is not relevant (skip)
- At rank 3: C is relevant, Precision@3 = 2/3 = 0.67
- At rank 4: D is not relevant (skip)
- At rank 5: E is relevant, Precision@5 = 3/5 = 0.6

AP = (1/3) * (1.0 + 0.67 + 0.6) = (1/3) * 2.27 = 0.76

### Implementation Notes

- If there are no relevant documents, AP is defined as 1.0
- Only contributes to the sum when a relevant document is found

## Hit Rate

### Formula

```
Hit Rate = 1 if |Retrieved intersection Relevant| > 0, else 0
```

### Explanation

Hit Rate is a binary metric that simply checks whether at least one relevant document was retrieved. It is the simplest retrieval metric.

### When to Use

- As a sanity check that the retrieval system is working at all
- When any relevant result is considered a success
- For very lenient evaluation where finding something is enough
- Useful for diagnostic purposes alongside more sophisticated metrics

### Example

Given:
- Retrieved documents: [A, B, C, D, E]
- Relevant documents: {C, F}

Calculation:
- C is in the retrieved set
- Hit Rate = 1

Another example:
- Retrieved: [A, B, D, E], Relevant: {C, F}
- Neither C nor F is retrieved
- Hit Rate = 0

### Implementation Notes

- Returns 1 or 0 (binary)
- Equivalent to checking if Recall@k > 0 for any k

## MetricsCalculator Class

The `MetricsCalculator` class provides a unified interface for computing all metrics and aggregating results across multiple test cases.

### Key Methods

**computeMetrics(retrieved, testCase)**
- Computes all retrieval metrics for a single query
- Returns precision@k, recall@k, MRR, nDCG@k, hit rate, and average precision
- Supports both binary and graded relevance

**aggregateMetrics(results)**
- Aggregates metrics across multiple test case results
- Computes mean, standard deviation, and 95th percentile for each metric
- Optionally groups results by category

### Statistical Aggregation

When evaluating a test suite, the calculator provides:
- **Mean**: Average value across all queries
- **Standard Deviation**: Measure of variance in metric values
- **95th Percentile**: Value below which 95% of queries fall

## Latency Metrics

The module also includes `calculateLatencyMetrics()` for measuring retrieval performance:

- **avgMs**: Average latency
- **medianMs**: Median latency (50th percentile)
- **p95Ms**: 95th percentile latency
- **p99Ms**: 99th percentile latency
- **minMs / maxMs**: Range of latencies
- **stdDevMs**: Standard deviation

## Quick Reference Table

| Metric | Range | Higher is Better | Considers Position | Graded Relevance |
|--------|-------|------------------|-------------------|------------------|
| Precision@k | 0-1 | Yes | No (top-k only) | No |
| Recall@k | 0-1 | Yes | No (top-k only) | No |
| MRR | 0-1 | Yes | Yes (first relevant) | No |
| nDCG@k | 0-1 | Yes | Yes (all positions) | Yes |
| Average Precision | 0-1 | Yes | Yes (all relevant) | No |
| Hit Rate | 0-1 | Yes | No | No |

## Observations

- [algorithm] Precision@k and Recall@k are complementary metrics measuring different aspects of retrieval quality #precision #recall
- [algorithm] MRR is optimal for single-answer queries where only the first relevant result matters #mrr
- [algorithm] nDCG is the most sophisticated metric, supporting graded relevance and position weighting #ndcg
- [algorithm] Average Precision provides a single-number summary that balances precision and recall #average-precision
- [fact] Standard k values are 1, 3, 5, 10, 20 for precision/recall and 5, 10, 20 for nDCG #implementation
- [fact] The MetricsCalculator class supports both binary and graded relevance judgments #implementation
- [insight] Hit Rate is useful as a baseline sanity check but insufficient for quality assessment #evaluation

## Relations

- implements [[RAG Evaluation Pipeline]] #evaluation
- documents [[indexer]] #indexer #metrics
- related_to [[Information Retrieval Theory]] #information-retrieval
- part_of [[DOC-002 RAG Evaluation Pipeline Documentation]] #team-doc-002