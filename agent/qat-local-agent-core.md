# QAT Local Agent Core

**Category:** agent
**Confidence:** production-proven
**Observed in:** [hf:unsloth/gemma-4-12B-it-qat-GGUF](https://huggingface.co/unsloth/gemma-4-12B-it-qat-GGUF)
**Compatible with:** python, typescript, any

## Problem
Running capable LLM agents locally or on cheap infra is blocked by VRAM/cost constraints, forcing dependence on expensive API calls for every agentic step including function-calling and legal document reasoning

## The Logic
Use QAT-quantized GGUF models (Q4_0 via llama-cpp-python or ollama) as the reasoning core for agentic loops. QAT preserves near-bfloat16 quality at 4-bit weight size, meaning a 12B model fits in ~8GB VRAM or CPU RAM. Wire native function-calling support into FastAPI tool-dispatch: model emits JSON tool call -> FastAPI route handles execution -> result injected back into context. Pattern: load_model(gguf_path) -> agent_loop(system_prompt, tools_schema) -> while not done: response = model.chat(messages) -> if response.tool_call: result = dispatch_tool(response.tool_call) -> messages.append(result) -> else: return response.content

## Steal This When
- Baby agent needs multi-step legal reasoning without per-token API costs
- Draft Terminal needs offline clause analysis with tool calls to LanceDB semantic search
- Greenlit needs local document classification before hitting paid APIs
- You need deterministic function-calling in an agentic loop you fully control

## Gotchas
- QAT Q4_0 still requires ~8GB RAM minimum for 12B - verify server spec before committing
- llama-cpp-python function-calling schema must match Gemma 4 chat template exactly or tool dispatch silently fails
- 256K context window advertised but practical limit with llama.cpp on CPU is much lower due to KV cache RAM growth - cap at 32K for legal docs
- GGUF quantization variants differ in quality vs speed - benchmark Q4_K_M vs Q4_0 for your specific legal text domain before locking in

## Real Implementation
https://huggingface.co/unsloth/gemma-4-12B-it-qat-GGUF
