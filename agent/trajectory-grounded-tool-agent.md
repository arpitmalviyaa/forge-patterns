# Trajectory-Grounded Tool Agent

**Category:** agent
**Confidence:** observed
**Observed in:** [hf:Jackrong/Qwopus3.6-27B-Coder-MTP-GGUF](https://huggingface.co/Jackrong/Qwopus3.6-27B-Coder-MTP-GGUF)
**Compatible with:** python, typescript, any

## Problem
LLM agents fail at multi-turn tool orchestration because they lack training on realistic tool-call/feedback loops, causing hallucinated tool signatures, broken retry logic, and shallow reasoning between steps

## The Logic
Fine-tune or prompt-engineer your agent loop using real agent trajectories that include: (1) tool definition schema, (2) the model's tool-call invocation, (3) environment/tool feedback, (4) model's next reasoning step conditioned on that feedback. The key insight is that the reasoning trace must be reconstructed end-to-end — not just input/output pairs — so the agent learns *how to think between tool calls*. In practice for FastAPI+LLM: structure your system prompt with explicit tool schemas, capture the full (thought -> tool_call -> tool_result -> thought) cycle in your message history, and treat each turn as a stateful reasoning checkpoint rather than a stateless completion. For legal-tech: tool_call=search_case_law(query) -> result -> reasoning about relevance -> next tool_call=extract_clause(doc_id).

## Steal This When
- your Baby agent is making tool calls but ignoring tool results in subsequent reasoning
- agent loops collapse after 2-3 turns because context isn't structured as trajectory
- you need reliable function-calling over legal document tools like search, extract, classify
- building a coding or document-drafting agent that must self-correct based on intermediate outputs

## Gotchas
- without trajectory structure in message history, model loses tool context after 3+ turns — always pass full (thought, call, result) triples not just results
- tool schema must be injected at system-prompt level consistently across every turn or model forgets available tools
- 27B GGUF is too heavy for free-tier inference — use smaller distilled or quantized variants for dev; pattern applies to any size
- off-thinking mode (no chain-of-thought tags) is faster but loses 8+ points on hard tasks — toggle based on task complexity

## Real Implementation
https://huggingface.co/Jackrong/Qwopus3.6-27B-Coder-MTP-GGUF
