# QAT GGUF Local Inference

**Category:** compression
**Confidence:** production-proven
**Observed in:** [hf:unsloth/gemma-4-26B-A4B-it-qat-GGUF](https://huggingface.co/unsloth/gemma-4-26B-A4B-it-qat-GGUF)
**Compatible with:** python, typescript, any

## Problem
Capable LLM with native function-calling is too large to run locally or on modest cloud hardware, blocking agentic workflows without expensive GPU instances

## The Logic
Use Quantization-Aware Training (QAT) GGUF checkpoints instead of post-hoc quantization: model is trained knowing it will be quantized, so Q4_0 weights retain near-bfloat16 quality. Load via llama-cpp-python or ollama in FastAPI, expose as OpenAI-compatible endpoint, wire tool/function-calling JSON schema to your existing agent loop. Pattern: model_path=gemma4-26B-Q4_0.gguf -> LlamaCpp(n_ctx=65536, n_gpu_layers=-1) -> chat_completion(tools=[your_schema]) -> parse tool_calls -> dispatch to FastAPI route handlers -> return result to model.

## Steal This When
- baby agent needs reliable function-calling without paying per-token API costs
- draft-terminal clause extraction needs offline processing for client confidentiality
- greenlit intake screening requires high-volume cheap inference on legal documents
- you want deterministic local tool dispatch without network latency or rate limits

## Gotchas
- QAT GGUF still requires ~14GB RAM for 26B Q4_0; verify hardware before committing architecture
- llama-cpp-python function-calling JSON schema parsing is stricter than OpenAI; validate schema round-trips before wiring to agent
- 256K context window claimed but practical throughput degrades sharply above 32K on consumer hardware; chunk legal docs accordingly
- MoE architecture means memory spikes during expert routing; set n_gpu_layers conservatively and benchmark before production

## Real Implementation
https://huggingface.co/unsloth/gemma-4-26B-A4B-it-qat-GGUF
