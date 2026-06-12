# Streamed Think Block Parsing

**Category:** pipeline
**Confidence:** production-proven
**Observed in:** [hf:JetBrains/Mellum2-12B-A2.5B-Thinking](https://huggingface.co/JetBrains/Mellum2-12B-A2.5B-Thinking)
**Compatible with:** python, typescript, any

## Problem
Agent outputs lack transparency; users cannot audit reasoning steps; debugging multi-step legal logic is opaque

## The Logic
Prompt the LLM to wrap all intermediate reasoning in <think>...</think> tags before emitting the final answer. In the FastAPI response handler, stream tokens and split on the closing </think> tag: buffer everything before it as internal_reasoning (store to LanceDB or log to Supabase), emit everything after it as the user-facing answer. This gives you a structured reasoning trace without a second API call. Pseudocode: for chunk in stream: if not past_think: think_buf += chunk.delta; if '</think>' in think_buf: past_think=True; final_buf += remainder else: final_buf += chunk.delta. On completion: save(think_buf) and return(final_buf).

## Steal This When
- you need auditable clause-by-clause reasoning in Draft Terminal
- you want to store reasoning traces in Supabase for compliance review
- a Baby agent step requires multi-hop legal logic that must be inspectable

## Gotchas
- Think block can be very long (up to model max); set a token budget or truncate before storing
- Some models emit </think> mid-stream inside a larger chunk; use a rolling buffer, not a simple split
- Temperature must stay ≥0.6 or reasoning collapses to shallow steps
- vLLM reasoning-parser flag required server-side; vanilla OpenAI SDK will not strip tags automatically

## Real Implementation
https://huggingface.co/JetBrains/Mellum2-12B-A2.5B-Thinking
