# SKILL: Pythonic Tool Call Chaining

Use this pattern when: LLM agents struggle to reliably chain multiple tool calls in sequence, producing malformed JSON or losing context between calls

**Category:** tool
**Confidence:** production-proven

## Core Logic
Define tools as JSON schema in system prompt. Model emits tool calls as a Python list between special delimiter tokens (<|tool_call_start|> ... <|tool_call_end|>). Execute each call, inject result under 'tool' role, then continue generation. This separates call syntax (Pythonic list) from result ingestion (role-tagged message), making multi-step chaining deterministic and parseable. Pattern: 1) system prompt with tools JSON array, 2) assistant emits [func(args)] between delimiters, 3) executor parses list, calls function, appends tool-role message with result, 4) model resumes to emit next call or final answer.

## When to Apply
- building a Baby agent that needs to chain legal document lookups, clause searches, and summarization in sequence
- you need deterministic tool call parsing without relying on JSON-mode which can hallucinate structure
- FastAPI endpoint needs to orchestrate multi-step LLM workflows where each step depends on prior tool output

## Avoid When
- tokenizer version drift breaks delimiter parsing - always pin tokenizer commit hash
- Pythonic list syntax means you must eval() or ast.literal_eval() carefully to avoid code injection - sanitize tool names and args against schema whitelist before execution
- tool role messages must be injected in exact conversation order or model loses chain-of-thought context
- temperature must stay low (0.2) or tool call syntax degrades - not suitable for creative generation modes

## Reference
[hf:LiquidAI/LFM2.5-8B-A1B](https://huggingface.co/LiquidAI/LFM2.5-8B-A1B)
