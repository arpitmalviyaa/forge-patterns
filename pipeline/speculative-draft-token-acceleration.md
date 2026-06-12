# Speculative Draft Token Acceleration

**Category:** pipeline
**Confidence:** production-proven
**Observed in:** [hf:unsloth/Qwen3.6-27B-MTP-GGUF](https://huggingface.co/unsloth/Qwen3.6-27B-MTP-GGUF)
**Compatible with:** python, typescript, any

## Problem
LLM inference latency is a bottleneck in agentic loops where the model must reason, call tools, and return structured output iteratively — each step blocking the next

## The Logic
Use Multi-Token Prediction (MTP) draft heads to speculatively generate 2+ tokens per forward pass, then verify in parallel. In practice: spin up llama-server with --spec-type draft-mtp --spec-draft-n-max 2, expose as OpenAI-compatible endpoint, call from FastAPI via httpx async client. Agent loop gets 1.5-2x throughput with zero accuracy loss. Pattern: slow_model_endpoint -> replace with mtp_server_endpoint, all downstream code unchanged because API contract is identical.

## Steal This When
- baby agent loop has >3 sequential LLM calls per user request and latency is visibly degrading UX
- draft-terminal clause generation pipeline has chained reasoning steps that feel sluggish
- you are self-hosting the model rather than using a cloud API and want to stretch hardware budget

## Gotchas
- MTP draft heads require matching llama.cpp build with CUDA support — CPU fallback is slower than vanilla inference if draft overhead exceeds savings
- np>1 (parallel slots) not yet supported with MTP so you cannot batch concurrent users through the same MTP server instance without a load balancer in front of multiple single-slot servers
- context window default is 262k tokens which will OOM on consumer GPUs — must set -c 8192 or lower explicitly
- OpenAI-compatible endpoint hides the MTP speedup from your code but you must pin the llama.cpp version since MTP flag names change between releases

## Real Implementation
https://huggingface.co/unsloth/Qwen3.6-27B-MTP-GGUF
