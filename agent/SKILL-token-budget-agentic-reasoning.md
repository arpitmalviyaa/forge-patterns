# SKILL: Token-Budget Agentic Reasoning

Use this pattern when: Agentic LLM loops burn excessive tokens on internal chain-of-thought, making multi-step legal-doc or code tasks prohibitively expensive and slow at inference time

**Category:** agent
**Confidence:** production-proven

## Core Logic
Train or select models that explicitly optimize 'thinking token efficiency' as a first-class metric alongside task accuracy. At runtime, enforce a soft token budget on the reasoning scratchpad (e.g., max_thinking_tokens=4096) before the model emits its final answer. If the model's CoT exceeds budget mid-task, checkpoint partial reasoning to a cheap key-value store (LanceDB metadata field), truncate, and resume from the checkpoint on the next agent loop iteration. Pseudocode: budget = 4096; tokens_used = 0; for step in agent_loop: result, thinking = model.step(state, max_thinking=budget - tokens_used); tokens_used += len(thinking); if tokens_used >= budget: store_checkpoint(lancedb, state, thinking); tokens_used = 0; state = load_checkpoint(lancedb)

## When to Apply
- Baby agent loops are hitting OpenAI/Anthropic token cost limits on multi-step legal document analysis
- Draft Terminal clause-expansion tasks require iterative reasoning but need predictable latency SLAs
- Greenlit screening pipelines run many sequential agentic checks and CoT verbosity compounds cost per submission

## Avoid When
- Truncating CoT mid-reasoning can cause the model to lose critical intermediate conclusions — always checkpoint reasoning state to LanceDB before truncating
- Token budget enforcement at the client layer only works if the model API exposes thinking tokens separately from output tokens (check provider support)
- 30% reduction is model-specific — measure your own baseline before committing budget constants in production code

## Reference
[hf:moonshotai/Kimi-K2.7-Code](https://huggingface.co/moonshotai/Kimi-K2.7-Code)
