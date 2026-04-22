---
title: graphrag Evaluation RAGAS Mapping
type: reference
permalink: agent-kb/projects/graphrag/research/graphrag-evaluation-ragas-mapping
created: 2026-03-29
modified: 2026-03-29
entity_type: reference
status: evergreen
tags:
- project/graphrag
- domain/evaluation
- domain/rag
- domain/research
---

# graphrag Evaluation RAGAS Mapping

How to implement RAGAS v0.4.3 metrics as `MetricCalculator` plugins in the graphrag evaluation system.

## MetricCalculator ABC (graphrag)

```python
# graphrag/interfaces/metric_calculator.py
class MetricCalculator(PluginBase):
    plugin_category = "metric_calculator"
    
    @abstractmethod
    async def calculate(self, sample: EvalSample) -> MetricResult: ...
```

`EvalSample` contains: `query`, `answer`, `retrieved_contexts`, `reference` (optional).
`MetricResult` contains: `metric_name`, `score: float`, `details: dict`.

## RAGAS v0.4.3 API

**CRITICAL: v0.3 → v0.4 breaking changes**:
- `evaluate()` → `single_turn_ascore()` (per-sample async evaluation)
- `Dataset` class removed → `SingleTurnSample` dataclass
- `contexts` field renamed → `retrieved_contexts`
- `EvaluationDataset` is now a list of `SingleTurnSample`

```python
# pip install ragas==0.4.3
from ragas.metrics import Faithfulness, AnswerRelevancy, ContextPrecision
from ragas import SingleTurnSample

sample = SingleTurnSample(
    user_input="What does InstrumentedStep do?",
    response="InstrumentedStep wraps plugins with retry and circuit breaker.",
    retrieved_contexts=[
        "InstrumentedStep adds retry logic...",
        "Circuit breaker prevents cascade failures...",
    ],
    reference="InstrumentedStep provides retry, circuit breaker, and OTel tracing.",  # optional
)

metric = Faithfulness()
score = await metric.single_turn_ascore(sample)   # returns MetricResult
print(score.score)  # float in [0, 1]
```

## Metric-to-Stage Mapping

| RAGAS Metric | Stage | Measures | Reference-free? |
|-------------|-------|----------|-----------------|
| `Faithfulness` | generation | Answer grounded in retrieved context | ✅ |
| `AnswerRelevancy` | generation | Answer addresses the query | ✅ |
| `ContextPrecision` | retrieval | Retrieved chunks are relevant | ✅ |
| `ContextRecall` | retrieval | Relevant content was retrieved | ❌ (needs reference) |
| `NoiseSensitivity` | retrieval→generation | Answer stability with noisy context | ✅ |
| `ResponseRelevancy` | generation | Full response relevance | ✅ |

## graphrag MetricCalculator Plugin Template

```python
from ragas.metrics import Faithfulness
from ragas import SingleTurnSample
from graphrag.interfaces import MetricCalculator, EvalSample, MetricResult

class RagasFaithfulnessCalculator(MetricCalculator):
    plugin_category = "metric_calculator"
    
    @property
    def name(self) -> str:
        return "ragas_faithfulness"
    
    @classmethod
    def config_schema(cls):
        class Config(BaseModel):
            llm_model: str = "gpt-4o-mini"
            batch_size: int = 10
        return Config
    
    async def initialize(self, config: dict) -> None:
        self._config = self.config_schema()(**config)
        self._metric = Faithfulness()
        # Optionally configure the LLM RAGAS uses internally
    
    async def calculate(self, sample: EvalSample) -> MetricResult:
        ragas_sample = SingleTurnSample(
            user_input=sample.query,
            response=sample.answer,
            retrieved_contexts=sample.retrieved_contexts,
            reference=sample.reference,  # may be None
        )
        result = await self._metric.single_turn_ascore(ragas_sample)
        return MetricResult(
            metric_name=self.name,
            score=result.score,
            details={"ragas_version": "0.4.3", "metric": "faithfulness"},
        )
```

## Integration with StageCompleted Events

The evaluation stage emits `MetricEmitted` events per metric. The AsyncEventBus routes them to:
1. Dashboard SSE stream (real-time display in TUI)
2. NDJSON log (`logs/pipeline-.ndjson`)
3. Optional: metrics store (Prometheus, if wired)

```python
# After generation stage completes
for metric_plugin in self.metric_calculators:
    sample = EvalSample.from_context(pipeline_ctx)
    result = await metric_plugin.calculate(sample)
    await bus.publish(MetricEmitted(
        stage="generation",
        metric_name=result.metric_name,
        score=result.score,
        details=result.details,
    ))
```

## Cost Profile

| Metric | LLM calls per sample | Cheap model sufficient? |
|--------|---------------------|------------------------|
| Faithfulness | 2 | ✅ (gpt-4o-mini) |
| AnswerRelevancy | 1 | ✅ |
| ContextPrecision | 1 | ✅ |
| ContextRecall | 2 | ✅ |
| NoiseSensitivity | 3 | ✅ |

At $0.15/1M input tokens (gpt-4o-mini), evaluating a 100-query batch with 5 metrics ≈ $0.10–0.50 total.

## Relations

- Implements → [[reference/rag/ragas-evaluation-standard]]
- Extends → [[projects/indexer/architecture/rag-evaluation-pipeline-overview]]
- Related → [[projects/graphrag/architecture/graphrag Plugin Categories Registry]]
- Related → [[projects/graphrag/research/graphrag SOTA RAG Landscape 2026]]

---
*Evolution Log*
- 2026-03-29: Initial creation from Layer 2 Agent C — RAGAS v0.4.3 API, MetricCalculator mapping, cost profile