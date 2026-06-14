# SKILL: Sparse Attention Long Context

Use this pattern when: Processing long legal documents (contracts, case files, regulatory texts) exhausts memory and compute—full attention over 1M tokens is quadratically expensive and practically unusable in a single inference pass

**Category:** context
**Confidence:** production-proven

## Core Logic
Instead of full dense attention across all tokens, partition the sequence into local windows plus a set of globally-attended 'anchor' tokens. Each token attends fully within its local window and to the global anchors, but skips distant non-anchor tokens. This reduces attention compute from O(n²) to roughly O(n * (window_size + num_anchors)). For legal-tech: treat section headers, clause boundaries, and party definitions as anchor tokens—they get global visibility while body text uses local attention. In LanceDB retrieval, pre-chunk by these anchor boundaries so that retrieved chunks map cleanly to the model's attention windows.

## When to Apply
- A legal document (contract, brief, regulation) exceeds 32K tokens and you need the model to reason over the whole thing in one shot
- You are building a clause-extraction or redline-comparison feature in Draft Terminal that must hold full contract context
- Memory OOM errors appear when naively feeding full documents to a transformer-based extraction pipeline
- You want to run a 'thinking' reasoning pass over an entire case file without chunking it into lossy fragments

## Avoid When
- Anchor token selection is critical—poor boundary detection means important cross-reference clauses get missed by local windows
- Most open-source models do NOT implement MSA natively; you must use vLLM/SGLang with MSA support or implement windowed attention yourself via FlashAttention sliding-window mode
- LanceDB chunk boundaries must align with attention window boundaries or retrieval results will straddle windows and confuse the model
- Thinking mode (chain-of-thought reasoning) consumes far more context budget than non-thinking mode—budget accordingly or switch modes per endpoint
- At 428B parameters this specific model is not indie-deployable; the PATTERN (sparse windowed attention with global anchors) is portable to smaller models like Mistral with sliding_window=4096

## Reference
[hf:MiniMaxAI/MiniMax-M3](https://huggingface.co/MiniMaxAI/MiniMax-M3)
