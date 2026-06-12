# QAT Local Inference Drop-in

**Category:** compression
**Confidence:** production-proven
**Observed in:** [hf:unsloth/gemma-4-12B-it-qat-GGUF](https://huggingface.co/unsloth/gemma-4-12B-it-qat-GGUF)
**Compatible with:** python, typescript, any

## Problem
Running a capable 12B+ instruction-tuned LLM locally or on a cheap VPS is blocked by VRAM/RAM ceiling; full bfloat16 weights are too large for indie-scale infra

## The Logic
Use Quantization-Aware Training (QAT) GGUF checkpoints instead of post-hoc quantized weights. QAT bakes quantization into the training loop so Q4_0 weights preserve near-bfloat16 quality. Pattern: (1) pull GGUF variant via llama-cpp-python or ollama, (2) expose via FastAPI endpoint with streaming, (3) swap in place of any OpenAI-compatible call in your existing code. No fine-tuning infra needed. Example: `llm = Llama(model_path='gemma-4-12B-it-qat-q4_0.gguf', n_ctx=32768, n_gpu_layers=-1)` then `llm.create_chat_completion(messages, stream=True)` behind a `/v1/chat/completions` FastAPI route.

## Steal This When
- you need a capable reasoning/coding LLM but cannot afford GPU API costs at scale
- Draft Terminal needs offline clause analysis without sending client data to third-party APIs
- Baby agent needs a local function-calling backbone that fits in 8-16GB RAM
- Greenlit needs a fast local classifier/reasoner during document ingestion pipelines
- you want to cut OpenAI spend by routing non-critical inference to a local model

## Gotchas
- QAT Q4_0 still needs ~8GB RAM minimum for 12B; confirm your VPS tier before committing
- GGUF tool-calling requires matching the exact chat template or function-call schema Gemma 4 expects — mismatched templates silently break agentic loops
- llama-cpp-python GPU offload requires CUDA build; default pip install is CPU-only and will be 10x slower
- 256K context window in spec but local GGUF inference with large contexts will OOM — cap n_ctx at 32K for safety
- Apache 2.0 license is clean for commercial use but verify Gemma 4 supplemental terms before deploying in legal-tech client-facing product

## Real Implementation
https://huggingface.co/unsloth/gemma-4-12B-it-qat-GGUF
