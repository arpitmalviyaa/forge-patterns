# SKILL: Configurable Reasoning Depth Toggle

Use this pattern when: Agent responses are either always slow/expensive (full chain-of-thought) or always shallow (no reasoning trace), with no way to tune cost vs quality per request type

**Category:** agent
**Confidence:** production-proven

## Core Logic
Expose a boolean flag (enable_thinking=True/False) at the inference call level that switches the model between full reasoning-trace mode and direct-answer mode. Route high-stakes requests (contract analysis, clause extraction, legal risk scoring) through thinking=True, and low-stakes requests (document classification, metadata tagging, simple Q&A) through thinking=False. Implement as a FastAPI dependency or middleware that inspects request payload fields like task_type or stakes_level and injects the correct flag before hitting the LLM endpoint. Example: if task.stakes in ['high','critical']: params['enable_thinking'] = True else: params['enable_thinking'] = False. Cache thinking=False responses aggressively in LanceDB; never cache thinking=True outputs as they depend on exact context state.

## When to Apply
- you have mixed-stakes legal document tasks where some need deep reasoning (contract risk) and others do not (document routing)
- LLM API costs are spiraling because all requests use full chain-of-thought
- you want to offer tiered response quality in Draft Terminal based on subscription or urgency level

## Avoid When
- thinking=False can silently produce plausible but wrong legal clause interpretations with no audit trail
- you must log which mode was used per request for compliance traceability
- switching modes mid-conversation breaks coherence; enforce mode consistency within a session
- caching thinking=True outputs risks serving stale legal reasoning on updated documents

## Reference
[hf:nvidia/NVIDIA-Nemotron-3-Ultra-550B-A55B-NVFP4](https://huggingface.co/nvidia/NVIDIA-Nemotron-3-Ultra-550B-A55B-NVFP4)
