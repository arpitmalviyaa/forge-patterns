# Sparse Expert Activation Routing

**Category:** compression
**Confidence:** production-proven
**Observed in:** [hf:deepseek-ai/DeepSeek-V4-Pro](https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro)
**Compatible with:** python, typescript, any

## Problem
LLM inference is too slow and memory-hungry for production legal-doc processing; loading a full dense model for every request wastes compute on irrelevant parameters

## The Logic
Use Mixture-of-Experts (MoE) architecture principle in your pipeline: partition capability into specialised sub-models or prompt-routed chains, activate only the relevant expert(s) per request. In practice for an indie stack: (1) define N specialised prompt templates or fine-tuned adapters (contract-review, clause-extraction, citation-check, summarisation); (2) add a lightweight router FastAPI endpoint that classifies incoming doc type and routes to the matching expert chain; (3) cache expert outputs in LanceDB keyed by (doc_type, content_hash); (4) only the activated expert consumes tokens/compute. Pseudocode: router = classify_doc(doc) -> expert_id; result = expert_registry[expert_id].run(doc); store(lancedb, key=(expert_id, hash(doc)), value=result)

## Steal This When
- you have multiple distinct legal doc types hitting the same endpoint
- inference costs are scaling linearly with doc volume
- different doc types need different reasoning depth (Flash vs Pro analogy)
- you want to avoid a monolithic prompt that tries to handle every clause type

## Gotchas
- router misclassification sends docs to wrong expert silently — always log routing decisions and add a fallback expert
- expert cache in LanceDB can go stale if prompt templates change — version your expert_id keys
- MoE savings only materialise when request volume is high enough to amortise routing overhead
- if using an external API (DeepSeek/OpenAI) instead of self-hosted, you cannot literally activate sparse weights — apply the pattern at the prompt/chain level only

## Real Implementation
https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro
