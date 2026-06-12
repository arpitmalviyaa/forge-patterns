# SKILL: Configurable Reasoning Mode Toggle

Use this pattern when: Agent LLM responses are either always verbose with chain-of-thought (slow, expensive) or always terse (low accuracy on complex tasks) — no runtime control over reasoning depth

**Category:** agent
**Confidence:** production-proven

## Core Logic
Expose a boolean flag (enable_thinking=True/False) in the chat template / system prompt or API call parameter that switches the model between full reasoning-trace mode and direct-answer mode. In code: if task.complexity == 'high' or task.type in ['legal_analysis','contract_review']: payload['enable_thinking'] = True else: payload['enable_thinking'] = False. Route through a FastAPI dependency that classifies request type before forwarding to the LLM endpoint, caching the classification result in LanceDB for similar future queries.

## When to Apply
- You have a legal document drafting endpoint where some requests need deep clause reasoning (enable_thinking=True) and others just need quick template fills (enable_thinking=False)
- Your FastAPI inference costs are high because every call uses full chain-of-thought even for trivial completions
- You want to give Draft Terminal users a 'Quick Draft' vs 'Deep Analysis' toggle in the UI without changing the underlying model

## Avoid When
- Reasoning mode adds significant latency and token cost — gate it behind task complexity scoring, not user preference alone
- If using a proxy/wrapper around a hosted model API, the flag may not pass through unless explicitly mapped in your request serializer
- Switching off reasoning for legal tasks risks missing edge-case clause conflicts — default to thinking=True for any contract or compliance route in Draft Terminal
- Caching reasoning-mode decisions in LanceDB by query embedding similarity can cause wrong mode selection if two queries are semantically similar but differ in required depth

## Reference
[hf:nvidia/NVIDIA-Nemotron-3-Ultra-550B-A55B-BF16](https://huggingface.co/nvidia/NVIDIA-Nemotron-3-Ultra-550B-A55B-BF16)
