# SKILL: Inline Control Token Injection

Use this pattern when: AI-generated text or speech outputs lack dynamic tone, style, or emotional modulation without rebuilding the entire prompt or switching models—responses feel flat, robotic, or contextually inappropriate

**Category:** pipeline
**Confidence:** production-proven

## Core Logic
Embed structured control tokens directly inside the content string rather than passing style/emotion as separate API parameters. Pattern: prefix or mid-sentence tags like <|emotion:determination|> or <|style:formal|> are parsed by the generation pipeline to shift output character at that exact position. In a FastAPI context: def inject_controls(text: str, segments: list[dict]) -> str: # segments = [{'pos': 42, 'token': '<|emotion:determination|>'}] for seg in sorted(segments, key=lambda x: x['pos'], reverse=True): text = text[:seg['pos']] + seg['token'] + text[seg['pos']:] return text. The LLM or generation model reads these inline markers as first-class tokens, not metadata, enabling per-sentence or per-clause behavioral shifts in a single inference pass.

## When to Apply
- Draft Terminal needs to vary legal document tone per clause—e.g. assertive in operative clauses, neutral in recitals
- Baby agent responses need confidence or urgency modulation without re-prompting
- Greenlit AI feedback needs to shift between encouraging and critical tones inline within a single response
- Any FastAPI endpoint generates structured text where different sections demand different register or style

## Avoid When
- Control tokens must be part of the model or parser vocabulary—injecting arbitrary tags into a standard LLM prompt does nothing unless the model was trained to recognize them; use prompt-level instructions as a fallback
- Token injection positions shift if upstream text is modified after tagging—always inject at render time, not storage time
- LanceDB stores raw text; store clean text, apply tokens at retrieval-to-generation boundary to avoid corrupting semantic embeddings
- Mid-utterance control creates jarring transitions if overused—limit to clause or sentence boundaries in legal-tech contexts

## Reference
[hf:bosonai/higgs-audio-v3-tts-4b](https://huggingface.co/bosonai/higgs-audio-v3-tts-4b)
