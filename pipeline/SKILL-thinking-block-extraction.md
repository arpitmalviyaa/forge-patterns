# SKILL: Thinking Block Extraction

Use this pattern when: Agent or LLM responses mix reasoning traces with final answers, making downstream parsing unreliable and bloating context windows

**Category:** pipeline
**Confidence:** production-proven

## Core Logic
Wrap chain-of-thought in a dedicated delimiter (e.g., <think>...</think>) then strip it before passing the final answer to downstream tools or storing in DB. At inference time: response = llm.complete(prompt); reasoning = extract_between(response, '<think>', '</think>'); answer = response.split('</think>')[-1].strip(). Store reasoning separately (e.g., Supabase audit log) if you need explainability, pass only answer to next pipeline stage.

## When to Apply
- your agent needs to show its work for legal explainability
- you want cheaper downstream calls by stripping verbose reasoning before vector embedding
- a clause-drafting step requires multi-step legal reasoning before emitting final contract text
- you need an audit trail of why a clause was generated without polluting the final document

## Avoid When
- model may emit partial or malformed think blocks — always guard with a fallback that returns full response if closing tag is missing
- reasoning tokens still cost compute even if stripped; budget accordingly
- stripping reasoning before embedding loses semantic signal — decide per use-case whether to embed reasoning, answer, or both
- temperature/top_p settings that work for instruct models may produce verbose or looping think blocks in reasoning models

## Reference
[hf:JetBrains/Mellum2-12B-A2.5B-Thinking](https://huggingface.co/JetBrains/Mellum2-12B-A2.5B-Thinking)
