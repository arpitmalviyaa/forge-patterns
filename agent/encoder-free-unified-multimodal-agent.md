# Encoder-Free Unified Multimodal Agent

**Category:** agent
**Confidence:** production-proven
**Observed in:** [hf:google/gemma-4-12B](https://huggingface.co/google/gemma-4-12B)
**Compatible with:** python, typescript, any

## Problem
Multimodal pipelines become brittle and slow when separate vision/audio encoders must be coordinated before reaching the LLM, adding latency, memory overhead, and separate fine-tuning passes for each modality

## The Logic
Instead of routing modalities through dedicated encoders (CLIP for images, Whisper for audio) before the LLM, project raw patches and waveforms directly into the LLM embedding space via lightweight linear layers, then route everything through a single decoder-only transformer. In a FastAPI context: receive multimodal payload -> linear_project(image_patches | audio_waveform | text_tokens) -> concat into unified token sequence -> single LLM forward pass -> structured JSON output via native function-calling. One model, one fine-tune, one inference endpoint.

## Steal This When
- Draft Terminal needs to ingest scanned legal PDFs (images) alongside text contracts in a single reasoning pass
- You want to avoid maintaining separate embedding pipelines for document images vs text in LanceDB
- A single FastAPI endpoint must handle mixed-modality legal documents without orchestrating multiple model calls
- You need to fine-tune on legal domain data once rather than separately tuning vision and text components

## Gotchas
- Encoder-free unified models are larger per modality than encoder-only alternatives at the same text quality; 12B may be too heavy for CPU-only Supabase edge functions
- Linear projection of raw patches loses spatial inductive bias that dedicated vision encoders provide; diagram-heavy legal docs may degrade vs CLIP-based approach
- 256K context fills fast when image patches are tokenized directly; a 20-page scanned contract can consume most of the window before text analysis begins
- Native function-calling schema must be validated carefully — Gemma 4 function-call format differs from OpenAI spec, requiring FastAPI middleware to normalise tool outputs before LanceDB writes

## Real Implementation
https://huggingface.co/google/gemma-4-12B
