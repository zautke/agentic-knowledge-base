---
title: RAGAS Evaluation Standard
type: reference
permalink: agent-kb/reference/rag/ragas-evaluation-standard
created: 2026-03-29
modified: 2026-03-29
entity_type: reference
status: evergreen
tags:
- project/agent-kb
- domain/rag
- domain/evaluation
- status/evergreen
---

# RAGAS Evaluation Standard

Global reference for the RAGAS (Retrieval-Augmented Generation Assessment) evaluation framework — the de-facto standard for component-level RAG evaluation without ground truth labels.

**Current stable version:** RAGAS v0.4.3 (released 2026-01-13)
**Paper:** arXiv 2309.15217

## Core Principle

RAGAS uses LLMs as judges internally — enabling **reference-free evaluation** (no pre-labeled QA pairs required). The LLM extracts claims, verifies them against context, reverse-engineers questions, etc.

## Complete Metric Catalog

### Reference-Free Metrics (No Ground Truth Needed)

| Metric | Stage | LLM-Based | Input Fields | What It Measures |
|--------|-------|-----------|--------------|------------------|
| **Faithfulness** | generation | YES | user_input, retrieved_contexts, response | Claim-level factual consistency vs context. Detects hallucination. |
| **Answer Relevancy** | generation | YES | user_input, response | Reverse-engineers questions; measures cosine sim to original. Detects off-topic/incomplete answers. |
| **Context Relevance** | retrieval | YES | user_input, retrieved_contexts | LLM judges utility of each retrieved sentence. Identifies noise. |
| **Semantic Similarity** | generation | NO | response | Embedding cosine similarity. Lightweight baseline. Zero LLM cost. |
| **String Presence** | generation | NO | response, target_strings | Regex/keyword match. |

### Requires Ground Truth

| Metric | Stage | Input Fields | What It Measures |
|--------|-------|--------------|------------------|
| **Context Precision** | retrieval | user_input, retrieved_contexts, reference | Are relevant docs ranked highest? |
| **Context Recall** | retrieval | user_input, retrieved_contexts, reference | What fraction of ground-truth info was retrieved? |
| **Answer Correctness** | generation | user_input, response, reference | Semantic match to ground-truth answer. |
| **BLEU/ROUGE/CHRF** | generation | response, reference | N-gram overlap metrics (classic NLP). |

### Advanced / Domain Metrics (v0.4+)

| Metric | Use Case |
|--------|----------|
| **AspectCritic** / **DiscreteMetric** | Custom LLM rubric evaluation. Define any aspect. |
| **Summarization Score** | Fluency, consistency, coherence for summaries. |
| **Tool Call Accuracy** | Agent tool invocation correctness. |

## SingleTurnSample Schema

```python
@dataclass
class SingleTurnSample:
    user_input: str                      # The query
    retrieved_contexts: list[str]         # Retrieved document texts
    response: str                         # Generated answer
    reference: str | None = None          # Ground truth answer (optional)
    reference_contexts: list[str] | None = None  # Relevant context list (optional)
    metadata: dict | None = None
```

## API Pattern (v0.4.3)

```python
from ragas.metrics.collections import Faithfulness, AnswerRelevancy
from ragas.llms import llm_factory
from openai import AsyncOpenAI

# Initialize metric with LLM judge
llm = llm_factory("gpt-4o-mini", client=AsyncOpenAI(api_key="..."))
metric = Faithfulness(llm=llm)

# Score a single sample (async)
sample = SingleTurnSample(
    user_input="What is photosynthesis?",
    retrieved_contexts=["Photosynthesis is..."],
    response="Plants convert sunlight..."
)
result = await metric.single_turn_ascore(sample)
score = result.value   # float 0-1
reason = result.reason # explanation string (LLM-generated)
```

## v0.3 → v0.4 Breaking Changes

| v0.3 | v0.4.3 |
|------|--------|
| `ground_truths` parameter | `reference` parameter |
| Returns raw float | Returns `MetricResult` object (`.value` for float) |
| `from ragas.metrics import Faithfulness` | `from ragas.metrics.collections import Faithfulness` |
| `AspectCritic` in collections | Use `@discrete_metric` decorator |
| `LangchainLLMWrapper` | Use `llm_factory()` |

## Recommended Metric Set (No Ground Truth)

Priority order for maximum signal with zero labels:

1. **Faithfulness** — hallucination detection; highest diagnostic value
2. **Answer Relevancy** — completeness/off-topic detection; orthogonal signal to Faithfulness
3. **Context Relevance** — retrieval noise filtering; retrieval-stage diagnostic
4. **Semantic Similarity** — lightweight embedding-based baseline; zero LLM cost

## Cost Profile

| Scenario | LLM Calls/Sample | Wall-clock | Tokens/Sample |
|----------|-----------------|-----------|---------------|
| Faithfulness only | 2-3 | ~1.5s | 2K-5K |
| Faithfulness + AnswerRelevancy + ContextRelevance | 5-7 | ~2s (parallel) | 5K-11K |
| Full set + SemanticSimilarity | 7-9 + 2 embeddings | ~2.5s | 7K-14K |

Monthly cost (10% sampling, 100K daily queries, GPT-4o-mini @ $0.075/1M): **~$8/month**.

## Inline vs Sampling Strategy

- **Development/debug**: Inline (every run). Accept 2-5s latency overhead.
- **Production**: Sample 10-20% of traffic. Evaluate hourly batch.
- **High-stakes queries**: Route to inline evaluation for all.

## MetricCalculator Integration Pattern

```python
class FaithfulnessMetricCalculator(MetricCalculator):
    plugin_category = "metric"
    
    @property
    def stage(self) -> str:
        return "generation"
    
    async def calculate(self, context: PipelineContext) -> MetricResult:
        hybrid = context.artifacts.get("hybrid_result")
        gen = context.artifacts.get("generator_result")
        
        retrieved_contexts = [vr.metadata.get("content", "") 
                               for vr in (hybrid.vector_results if hybrid else [])]
        
        sample = SingleTurnSample(
            user_input=context.query,
            retrieved_contexts=retrieved_contexts,
            response=gen.text if gen else "",
        )
        
        ragas_result = await self.metric.single_turn_ascore(sample)
        return MetricResult(
            name="faithfulness",
            value=float(ragas_result.value or 0.0),
            stage="generation",
            metadata={"reason": ragas_result.reason},
            timestamp=datetime.utcnow(),
        )
```

## Relations

- part_of [[Hybrid RAG Architecture – SOTA 2025-2026]]
- related_to [[RAG Evaluation Pipeline - Module Overview]]
- related_to [[RAG Evaluation Metrics Reference]]
- implemented_by [[graphrag Evaluation RAGAS Mapping]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-03-29 | Initial creation | Layer 2-C research: RAGAS v0.4.3 deep-dive | graphrag plugin implementation sprint |
| 2026-05-28 | Repaired two broken relation links: `[[RAG Evaluation Pipeline Overview]]` → `[[RAG Evaluation Pipeline - Module Overview]]` and `[[graphrag: Evaluation RAGAS Mapping]]` → `[[graphrag Evaluation RAGAS Mapping]]` (matched to real note titles) | Links never resolved | KB curation hygiene sweep (operations/reference partition) |