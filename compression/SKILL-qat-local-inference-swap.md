# SKILL: QAT Local Inference Swap

Use this pattern when: Cloud LLM API costs scale with usage, latency spikes under load, and sensitive legal data cannot be sent to third-party endpoints

**Category:** compression
**Confidence:** production-proven

## Core Logic
Use Quantization-Aware Training (QAT) GGUF models instead of naive post-training quantization. QAT bakes quantization error correction into training, so Q4_0 weights behave near-identically to bfloat16 at runtime. Pattern: (1) swap cloud LLM call with local llama-cpp-python or ollama endpoint serving GGUF, (2) expose identical OpenAI-compatible /v1/chat/completions interface via FastAPI proxy, (3) client code (Next.js or Python) unchanged. For 12B model: fits in ~8GB VRAM or ~16GB unified RAM, enabling MacBook Pro or budget GPU server deployment. Legal docs never leave the machine.

## When to Apply
- legal document drafting requires data residency or cannot use cloud APIs
- API cost per-token is becoming non-trivial during Draft Terminal document generation loops
- you need function-calling and agentic tool-use locally without OpenAI dependency
- Baby agent needs a cheap local reasoning backbone for document classification tasks

## Avoid When
- QAT GGUF requires llama-cpp-python built with CUDA/Metal support—pip install alone is insufficient
- 256K context window on 12B is theoretical; practical usable context under 8GB VRAM is ~32K before memory pressure degrades throughput
- function-calling schema must match Gemma 4 chat template exactly—OpenAI tool_calls format needs adapter layer
- cold start on first request is slow (~3-8s model load); keep process warm with FastAPI lifespan event
- audio/video multimodal features in 12B unified model are not yet supported in most GGUF runtimes—text+image only

## Reference
[hf:google/gemma-4-12B-it-qat-q4_0-gguf](https://huggingface.co/google/gemma-4-12B-it-qat-q4_0-gguf)
