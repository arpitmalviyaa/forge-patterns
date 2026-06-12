# Agentic Tool-Use Chat Template

**Category:** agent
**Confidence:** production-proven
**Observed in:** [hf:CohereLabs/North-Mini-Code-1.0](https://huggingface.co/CohereLabs/North-Mini-Code-1.0)
**Compatible with:** python, any

## Problem
LLM agents need structured, reliable tool invocation without brittle prompt engineering or custom parsing logic

## The Logic
Define tools as JSON schema objects with name/description/parameters. Pass them via tokenizer.apply_chat_template(messages, tools=tools) so the model natively formats tool calls and results in a standardized turn structure. The model emits structured tool-call tokens; parse them, execute the real function, append the result as a tool-role message, and continue generation in a loop until the model emits a final answer with no pending tool calls. This eliminates custom regex parsing and works reliably across agentic loops.

## Steal This When
- building a Baby agent loop that calls FastAPI endpoints, LanceDB queries, or Supabase mutations as tools
- you want the LLM to decide when to search vs answer vs write code
- replacing fragile string-matched tool dispatch with schema-driven native tool calling

## Gotchas
- Chat template format varies per model family—always use the model's own tokenizer.apply_chat_template, never hand-roll the tool schema injection
- Tool result must be appended as a message with role='tool' and matching tool_call_id or the model loses context
- temperature=1.0 + top_p=0.95 recommended for agentic tasks; lower temperature causes repetitive tool loops
- 256K context fills fast in multi-turn agentic sessions—prune or summarize old tool results

## Real Implementation
https://huggingface.co/CohereLabs/North-Mini-Code-1.0
