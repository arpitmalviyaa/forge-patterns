# Parallel Coordinate Decoding Pattern

**Category:** pipeline
**Confidence:** production-proven
**Observed in:** [hf:nvidia/LocateAnything-3B](https://huggingface.co/nvidia/LocateAnything-3B)
**Compatible with:** python, typescript, any

## Problem
Extracting structured spatial or positional data (bounding boxes, date ranges, clause spans, form field locations) from LLM/VLM outputs is slow because autoregressive token-by-token decoding generates each coordinate sequentially, creating latency that compounds when processing many documents or UI elements

## The Logic
Instead of decoding structured output fields token-by-token in sequence, predict all fields of a structured record in a single parallel step. For bounding boxes: predict [x1, y1, x2, y2] simultaneously rather than x1→y1→x2→y2. Transfer to legal-tech: when extracting clause spans, date ranges, party names, or form field coordinates from documents, define a fixed-schema output head that emits all fields of a record in one forward pass. Implementation sketch: define a Pydantic model with all target fields, use structured outputs / function-calling / constrained decoding (e.g. Outlines or instructor library) to force the LLM to emit the complete record atomically. In FastAPI: accept document chunk, call LLM with schema-constrained output, receive complete extraction record, store all fields to LanceDB in one write. No sequential follow-up calls per field. Key insight: geometric/positional consistency is preserved because all coordinates are predicted with shared context rather than each conditioned on previous tokens.

## Steal This When
- extracting multiple structured fields from a legal document chunk in one LLM call
- building a document annotation pipeline where clause boundaries or date spans must be extracted at scale
- Draft Terminal needs to locate and highlight specific contract provisions with coordinates or character offsets for PDF rendering
- any pipeline where you currently make N sequential LLM calls to extract N fields from the same context window

## Gotchas
- Parallel decoding requires the model to support structured/constrained output natively or via a wrapper—vanilla chat completions may still decode sequentially internally
- Schema must be fully defined upfront; dynamic or open-ended field sets break the pattern and force fallback to sequential extraction
- Field interdependencies (e.g. end_date must be after start_date) are not enforced by parallel decoding—add a post-validation layer
- NVIDIA license is non-commercial so LocateAnything-3B itself cannot be used in production; the pattern is the steal, not the model weights
- Throughput gains only materialize when batching many records—single-document latency improvement may be negligible

## Real Implementation
https://huggingface.co/nvidia/LocateAnything-3B
