# SKILL: Inline Control Token Injection

Use this pattern when: Generated text or audio output lacks contextual tone variation, making AI responses feel robotic or emotionally flat even when the underlying content demands different registers (urgent legal deadline vs. reassuring contract summary)

**Category:** pipeline
**Confidence:** production-proven

## Core Logic
Embed structured control tokens directly inside the content string at authoring or generation time rather than as separate API parameters. Pattern: '<|category:value|> text segment <|category:other_value|> next segment'. At render/synthesis time, a downstream processor strips tokens and applies the named modifier to the bracketed segment. In a text pipeline this means: 1) define a token schema {category: [allowed_values]}, 2) inject tokens at semantic boundaries in the output string, 3) strip tokens before display but use them to drive CSS class, voice style, or emphasis in the rendered output. Example for Draft Terminal: LLM output includes '<|urgency:high|> This clause expires in 3 days. <|urgency:normal|> The remaining sections are standard boilerplate.' Frontend parses tokens, renders high-urgency spans in red with alert icon, normal spans in default style. Same string works as plain text if token-stripping fallback fires.

## When to Apply
- LLM output needs to carry rendering or tone metadata without a separate structured field
- you want a single string to be both human-readable fallback and machine-parseable for styled rendering
- legal documents need urgency or risk-level annotation inline without breaking the prose
- building a voice layer on top of existing text agent output

## Avoid When
- Token schema must be strictly validated server-side before storage in Supabase or LanceDB to prevent injection via crafted input
- LLM must be explicitly prompted with the token schema or it will hallucinate token syntax inconsistently
- Token stripping regex must be applied before any embedding generation or the control tokens pollute vector semantics
- If tokens span chunk boundaries during RAG retrieval the modifier context is lost, so chunk at token boundaries not arbitrary character counts

## Reference
[hf:bosonai/higgs-audio-v3-tts-4b](https://huggingface.co/bosonai/higgs-audio-v3-tts-4b)
