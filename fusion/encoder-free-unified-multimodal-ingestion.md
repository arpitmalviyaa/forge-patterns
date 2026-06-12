# Encoder-Free Unified Multimodal Ingestion

**Category:** fusion
**Confidence:** production-proven
**Observed in:** [hf:google/gemma-4-12B](https://huggingface.co/google/gemma-4-12B)
**Compatible with:** python, any

## Problem
Adding multimodal support (images, documents, audio) to an existing LLM pipeline requires wiring separate encoder models, managing multiple inference sessions, versioning separate model weights, and stitching outputs together before the main LLM sees them — causing latency, deployment complexity, and fine-tuning friction.

## The Logic
Instead of routing each modality through a dedicated encoder (CLIP for images, Whisper for audio, etc.) and concatenating embeddings, project raw modality tokens (image patches as flat pixel grids, audio as waveforms) through a single lightweight linear layer directly into the LLM's embedding space. All modalities then enter the same decoder-only transformer as token sequences. Pattern: raw_input -> linear_projection(modality) -> token_sequence -> concat([text_tokens, modality_tokens]) -> single_transformer_forward_pass. In FastAPI terms: one /infer endpoint accepts multipart form with text + optional image bytes; a single model.generate() call handles all inputs without branching logic per modality.

## Steal This When
- Draft Terminal needs to ingest uploaded contract PDFs or scanned images alongside text prompts without maintaining a separate OCR/vision pipeline
- you want to fine-tune the whole ingestion pipeline end-to-end on legal documents without frozen encoder weights blocking gradient flow
- reducing cold-start latency on a FastAPI endpoint that currently loads 2-3 models at startup

## Gotchas
- Linear projection layers must be trained on domain-specific data (legal docs) to produce useful token representations — a generic checkpoint may project legal scan patches poorly
- Without a dedicated vision encoder, image resolution handling becomes your responsibility: padding and patch-size normalisation must be done client-side before posting to the API
- 256K context window sounds generous but a single high-res contract scan as raw patches can consume 10-40K tokens, leaving less room for chain-of-thought reasoning in the same pass
- Encoder-free models are harder to swap modality components on independently — you cannot upgrade just the image understanding without retraining the whole model

## Real Implementation
https://huggingface.co/google/gemma-4-12B
