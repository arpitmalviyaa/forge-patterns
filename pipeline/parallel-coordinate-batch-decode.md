# Parallel Coordinate Batch Decode

**Category:** pipeline
**Confidence:** production-proven
**Observed in:** [hf:nvidia/LocateAnything-3B](https://huggingface.co/nvidia/LocateAnything-3B)
**Compatible with:** python, typescript, any

## Problem
Autoregressive token-by-token decoding of structured outputs (bounding boxes, clause spans, date ranges, entity coordinates) creates serial bottlenecks—each token waits for the previous, multiplying latency by the number of tokens in the structure

## The Logic
Instead of decoding structured multi-value outputs sequentially (token1 -> token2 -> token3 -> token4 for x1,y1,x2,y2), predict all values in a single parallel forward pass by treating the full structure as a fixed-width output head. For legal-tech: when extracting clause boundaries, party names, or obligation spans from a document, define a fixed schema (start_char, end_char, confidence, label) and decode all fields simultaneously via parallel classification/regression heads rather than generating them token-by-token. In FastAPI: run one model call that returns N structured objects in parallel rather than N sequential decode loops. Core pseudocode: output = model.parallel_heads(hidden_state) -> {field_1: tensor, field_2: tensor, ...} unpacked in one shot, versus autoregressive: for field in fields: output[field] = model.next_token(output[field-1]).

## Steal This When
- you are extracting fixed-schema structured data from documents and LLM decoding latency is the bottleneck
- Draft Terminal needs to highlight multiple clause spans simultaneously rather than streaming them one at a time
- you have a known output schema (e.g. obligation: {party, action, deadline, condition}) and are paying autoregressive cost per field
- batch processing many contracts where per-document decode time compounds

## Gotchas
- only works when output schema is fixed-width and known ahead of time—open-ended generation still needs autoregressive
- requires fine-tuning or a model already trained with parallel heads; off-the-shelf LLMs default to autoregressive
- geometric consistency constraints (e.g. x1 < x2) must be enforced post-decode since parallel heads have no inter-field attention during generation
- for legal span extraction, character offsets predicted in parallel may have higher variance than sequential—add a calibration/snapping step to sentence boundaries
- NVIDIA license is non-commercial so you cannot deploy this model directly—pattern is transferable but model is not

## Real Implementation
https://huggingface.co/nvidia/LocateAnything-3B
