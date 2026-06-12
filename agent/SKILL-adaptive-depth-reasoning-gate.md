# SKILL: Adaptive Depth Reasoning Gate

Use this pattern when: Agent wastes tokens doing deep chain-of-thought on trivial steps, or conversely rushes critical decisions without sufficient reasoning, causing either high latency or poor task reliability in long-horizon workflows

**Category:** agent
**Confidence:** production-proven

## Core Logic
Before each agent action, classify the decision into complexity tiers (trivial/moderate/critical). For trivial actions (deterministic lookups, CRUD ops, formatting), execute immediately with no reasoning chain. For moderate actions (ambiguous user intent, multi-step logic), apply lightweight scratchpad. For critical actions (irreversible writes, legal clause generation, schema migrations), invoke full chain-of-thought with explicit verification step before execution. Pseudocode: def decide_thinking_depth(action, context): complexity = classify_action(action, context) if complexity == 'trivial': return execute_direct(action) elif complexity == 'moderate': return execute_with_scratchpad(action) else: reasoning = full_cot_with_verification(action, context) if reasoning.verified: return execute_with_audit_log(action, reasoning) else: return request_human_confirmation(action, reasoning)

## When to Apply
- Draft Terminal agent needs to generate legal clauses where some are boilerplate (trivial) and others require careful reasoning about jurisdiction or risk
- Baby agent routing tasks of varying complexity to avoid uniform CoT overhead on every FastAPI call
- Any agentic pipeline where LLM inference cost and latency matter but correctness on high-stakes steps is non-negotiable

## Avoid When
- Misclassifying a critical action as trivial is catastrophic in legal-tech — always err toward higher reasoning tier when uncertain
- Classification logic itself must be cheap; avoid calling LLM to decide thinking depth or you negate the benefit
- In streaming FastAPI responses, deep CoT adds latency the user sees — expose reasoning depth as a user-tunable parameter
- LanceDB vector similarity scores alone are insufficient to classify action criticality — combine with rule-based checks on action type

## Reference
[hf:nex-agi/Nex-N2-mini](https://huggingface.co/nex-agi/Nex-N2-mini)
