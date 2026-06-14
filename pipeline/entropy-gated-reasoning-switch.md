# Entropy-Gated Reasoning Switch

**Category:** pipeline
**Confidence:** observed
**Observed in:** [hf:prefeitura-rio/Rio-3.5-Open-397B](https://huggingface.co/prefeitura-rio/Rio-3.5-Open-397B)
**Compatible with:** python, typescript, any

## Problem
LLM agent wastes tokens on verbose chain-of-thought for easy steps, yet under-thinks hard steps, causing both cost blowout and accuracy failures in the same workflow

## The Logic
Monitor per-token entropy of the model output distribution at each reasoning step. If entropy is trending UP (model is uncertain, exploring), suppress token emission and let reasoning continue in compressed/latent form (e.g. skip verbose scratchpad, use a tighter prompt, or route to a cheaper internal loop). If entropy is trending DOWN (model is converging, confident), switch back to full explicit token output to commit the answer. In practice for Python/FastAPI: wrap LLM calls with a streaming entropy tracker; if rolling entropy delta > threshold, switch to a 'silent reasoning' prompt variant that returns only the conclusion; if entropy is stable/falling, allow full chain-of-thought. This creates a dynamic budget allocator that spends tokens only where uncertainty demands them.

## Steal This When
- Baby agent is burning tokens on multi-step legal clause analysis where some steps are trivially obvious and some are genuinely ambiguous
- Draft Terminal needs to decide whether to show full reasoning trace to user or just the conclusion based on how confident the model is
- Any agentic loop has unpredictable token costs that blow the per-request budget

## Gotchas
- Entropy estimation requires access to logprobs, which some hosted APIs do not expose — verify your LLM provider supports it before designing around this
- Switching too aggressively between modes can fragment coherent reasoning chains and produce incoherent outputs; set a minimum dwell time per mode
- Entropy trends are noisy for short sequences; compute over a rolling window of at least 5-10 tokens to avoid false switches
- Latent/silent mode needs careful prompt engineering so the model does not interpret silence as instruction to hallucinate a short answer without reasoning

## Real Implementation
https://huggingface.co/prefeitura-rio/Rio-3.5-Open-397B
