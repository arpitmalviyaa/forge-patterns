# Native Function-Call Agent Loop

**Category:** agent
**Confidence:** production-proven
**Observed in:** [hf:google/gemma-4-12B-it](https://huggingface.co/google/gemma-4-12B-it)
**Compatible with:** python, typescript

## Problem
LLM agents require complex glue code to parse tool calls, handle multi-turn agentic loops, and integrate structured outputs — brittle prompt engineering breaks when model versions change

## The Logic
Use a model with native function-calling baked into its instruction-tuned format rather than prompt-hacked tool use. Pattern: (1) Define tools as typed JSON schemas in the system prompt using the model's native `system` role. (2) Model emits structured tool-call tokens that parse deterministically — no regex needed. (3) FastAPI route receives tool-call payload, executes against LanceDB/Supabase, returns structured result. (4) Feed result back into context as a `tool` role message. (5) Model decides whether to call another tool or emit final answer. Loop terminates on final-answer token or max-turns guard. Pseudocode: tools=[{name:'search_clauses', params:{query:str, jurisdiction:str}}]; response=model.chat(messages, tools=tools); while response.tool_call: result=execute_tool(response.tool_call); messages.append({role:'tool', content:result}); response=model.chat(messages, tools=tools); return response.text

## Steal This When
- building Baby agent tool-dispatch loop
- Draft Terminal needs to call clause-search or jurisdiction-lookup tools mid-generation
- Greenlit requires multi-step reasoning over legal documents with structured tool outputs
- replacing fragile regex-based tool parsing in existing agent code

## Gotchas
- 256K context fills fast in multi-turn agentic loops — implement sliding-window message pruning that preserves tool schemas and final results but drops intermediate reasoning
- Encoder-free unified architecture means image/doc inputs go in as raw patches — PDF page images need chunking before injection, not pre-encoded embeddings
- Native function-calling schema must match model's expected format exactly — Gemma 4 uses a specific JSON schema dialect, test schema validation before production
- 12B model needs ~8GB VRAM minimum — on constrained infra fall back to E4B variant with same function-calling capability but 128K context limit

## Real Implementation
https://huggingface.co/google/gemma-4-12B-it
