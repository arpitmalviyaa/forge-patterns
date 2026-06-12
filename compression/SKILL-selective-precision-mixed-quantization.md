# SKILL: Selective Precision Mixed Quantization

Use this pattern when: LLM inference is too slow or memory-hungry for production serving, but blanket quantization degrades reasoning quality on complex tasks like legal analysis or code generation

**Category:** compression
**Confidence:** production-proven

## Core Logic
Identify which model components tolerate low-precision best (MoE experts, feed-forward layers) vs. which are quality-sensitive (attention projections, layer norms). Quantize only the tolerant components to FP4/INT4 while keeping sensitive components at BF16/FP16. Apply QAT (quantization-aware training) or GPTQ calibration on the tolerant layers. Pattern: for each layer, if layer_type in TOLERANT_TYPES: quantize(layer, bits=4, block_size=32) else: keep_original_precision(layer). Result: memory footprint drops ~50% with near-zero quality loss on reasoning benchmarks.

## When to Apply
- serving a large local model via FastAPI and hitting OOM or latency SLA failures
- deploying a reasoning model for legal clause analysis where accuracy cannot degrade but cost must drop
- choosing between full-precision slow inference and quantized fast inference for Draft Terminal's LLM backend

## Avoid When
- Blanket INT4 quantization on attention layers causes noticeable degradation on multi-step legal reasoning chains — always keep attention at higher precision
- Block size matters: block_size=32 is safer than 64+ for legal/code domains where token semantics are dense
- QAT requires fine-tuning access; if using third-party model via API this pattern is not applicable — use it only when you control the model weights
- Verify quality on your actual domain (legal clauses, contract extraction) not just general benchmarks before shipping

## Reference
[hf:XiaomiMiMo/MiMo-V2.5-Pro-FP4-DFlash](https://huggingface.co/XiaomiMiMo/MiMo-V2.5-Pro-FP4-DFlash)
