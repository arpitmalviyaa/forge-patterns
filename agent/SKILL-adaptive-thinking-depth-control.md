# SKILL: Adaptive Thinking Depth Control

Use this pattern when: Agent wastes tokens and latency doing deep chain-of-thought reasoning on trivial sub-steps, but also shallow-reasons on critical decisions that need careful planning—no mechanism to calibrate thinking depth per action

**Category:** agent
**Confidence:** production-proven

## Core Logic
Before executing any agent action, classify the action into a complexity tier (trivial|routine|critical) based on reversibility, downstream dependency count, and ambiguity score. Trivial actions (e.g., read file, format string) execute immediately with no scratchpad. Routine actions (e.g., API call, DB query) get a short plan check. Critical actions (e.g., write contract clause, submit legal filing, delete record) trigger full chain-of-thought with explicit assumption listing and a self-verification pass before execution. In code: if action.tier == 'trivial': execute(); elif action.tier == 'routine': think_briefly(); execute(); else: think_deeply(); verify(); execute(). This keeps p50 latency fast while protecting p99 correctness on high-stakes steps.

## When to Apply
- Baby agent is calling an LLM for every micro-step and burning tokens on trivial actions like formatting or lookups
- Agent makes confident errors on high-stakes legal steps (clause generation, jurisdiction checks) because it treated them like routine tasks
- You need to reduce median latency without degrading accuracy on the steps that matter most
- Draft Terminal agent must decide quickly whether a user instruction is a safe cosmetic edit vs a structural contract change requiring review

## Avoid When
- Tier classification itself can be wrong—a misclassified 'trivial' action on a legal document is worse than always using deep reasoning; start with conservative thresholds biased toward 'critical'
- Tier classification adds a small latency overhead; cache classifications for repeated action types in LanceDB to amortize cost
- Models without explicit thinking-budget control (not Qwen3.5-scale) may ignore soft instructions to 'think briefly'—use token limits or stop sequences as hard enforcement
- In legal-tech context, default almost everything to 'critical' tier until you have logged evidence that a class of actions is genuinely safe to shortcut

## Reference
[hf:nex-agi/Nex-N2-Pro](https://huggingface.co/nex-agi/Nex-N2-Pro)
