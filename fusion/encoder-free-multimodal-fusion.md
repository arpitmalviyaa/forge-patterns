# Encoder-Free Multimodal Fusion

**Category:** fusion
**Confidence:** production-proven
**Observed in:** [hf:google/gemma-4-12B-it](https://huggingface.co/google/gemma-4-12B-it)
**Compatible with:** python, any

## Problem
Multimodal pipelines (text + image + audio) require orchestrating multiple separate encoder models, increasing latency, deployment complexity, memory footprint, and making end-to-end fine-tuning across modalities impractical

## The Logic
Instead of routing each modality through a dedicated encoder (vision encoder → embeddings, audio encoder → embeddings, then concat → LLM), project raw modality tokens (image patches, audio waveforms) directly into the LLM embedding space via lightweight linear projection layers, then process everything in a single decoder-only transformer. One model, one forward pass, one embedding space. Pseudocode: raw_image_patches → linear_proj → token_embeddings; raw_audio_frames → linear_proj → token_embeddings; interleave with text_token_embeddings → unified_decoder(all_tokens). This collapses N-model orchestration into 1-model inference.

## Steal This When
- Draft Terminal needs to ingest uploaded contract PDFs rendered as images alongside extracted text in a single reasoning pass
- You want to avoid running a separate OCR service plus a separate text model plus a separate image model in sequence
- Legal documents arrive as scanned image PDFs where layout context matters and text extraction alone loses structural meaning
- You need end-to-end fine-tuning on legal domain data across text and image modalities without managing gradient flow across separate encoder/decoder boundaries

## Gotchas
- Encoder-free models push modality understanding burden entirely onto the decoder, requiring larger decoder capacity — 12B params minimum for reliable multimodal reasoning, smaller is risky
- Raw patch projection means image understanding quality degrades on very high resolution or complex diagrams compared to dedicated vision encoders at the same effective parameter count
- 256K context window sounds large but image patches are token-hungry — a single full-page scanned document can consume 2K-8K tokens depending on resolution, budget carefully per legal document page
- FastAPI streaming responses with multimodal inputs require careful async handling since image tokenization is not trivially streamable — pre-tokenize images before entering the stream endpoint
- LanceDB vector storage for multimodal embeddings requires a unified embedding model that also understands images — your retrieval layer must match modality assumptions of your generation model or you get retrieval-generation modality mismatch

## Real Implementation
https://huggingface.co/google/gemma-4-12B-it
