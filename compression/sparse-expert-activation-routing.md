# Sparse Expert Activation Routing

**Category:** compression
**Confidence:** production-proven
**Observed in:** [hf:deepseek-ai/DeepSeek-V4-Flash](https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash)
**Compatible with:** python, typescript, any

## Problem
LLM inference is too slow or expensive for real-time legal document analysis and agentic workflows because the full model is activated for every token

## The Logic
Use Mixture-of-Experts (MoE) architecture pattern: define N total expert sub-networks but only activate K of them per token via a learned router. Applied to API design: route each incoming request/document chunk to specialized sub-handlers (contract expert, clause expert, citation expert) rather than running all logic for every input. In practice for indie stack: decompose a monolithic FastAPI LLM endpoint into routed specialist chains where a lightweight classifier (the router) selects which RAG pipeline + prompt template + LanceDB collection to invoke. Total params = N * expert_size, activated params = K * expert_size where K << N. This gives sub-linear cost scaling as you add more legal domain specializations.

## Steal This When
- you have multiple distinct legal document types (contracts, filings, briefs) and want to avoid one giant prompt-and-RAG chain handling all of them
- latency is a bottleneck on the FastAPI layer and you want to skip irrelevant retrieval collections in LanceDB
- you are scaling Baby agent to handle multiple task types without paying full inference cost per task
- Draft Terminal needs to route clause analysis vs citation lookup vs formatting without sequential chaining

## Gotchas
- Router classifier itself adds latency if over-engineered; keep it lightweight (keyword heuristic or tiny embedding classifier, not another LLM call)
- Expert imbalance: some legal doc types are rare so experts for them under-train; compensate with synthetic data or shared base prompts
- MoE pattern at the model level requires specific inference infrastructure (vLLM with expert parallelism); at the application layer it is just conditional routing which is trivially implementable
- FP4 quantization used here means you cannot self-host without specific GPU support; prefer API access (DeepSeek API or OpenRouter) and apply the routing pattern at your application layer instead

## Real Implementation
https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash
