# SKILL: Configurable Reasoning Toggle Pattern

Use this pattern when: LLM responses are either always verbose with chain-of-thought traces (slow, expensive) or always terse (low accuracy on complex tasks) — no runtime control over reasoning depth

**Category:** agent
**Confidence:** production-proven

## Core Logic
Expose a boolean flag (e.g. enable_thinking=True/False) injected into the chat template/system prompt at request time. When True, the model emits a scratchpad reasoning trace before the final answer. When False, it skips directly to the answer. In your FastAPI layer: detect task complexity (e.g. clause extraction vs. simple lookup), set the flag accordingly, strip the reasoning trace from the response before returning to the client unless the caller explicitly requests it. Pseudocode: thinking = task.complexity > THRESHOLD or request.debug_mode; payload = build_prompt(messages, enable_thinking=thinking); raw = llm.complete(payload); answer = raw.split('<final_answer>')[1] if not request.include_trace else raw

## When to Apply
- legal clause analysis needs deep reasoning but document classification does not
- you want to expose a debug/trace mode to pro users in Draft Terminal
- latency SLAs differ across endpoints and you need one model to serve both

## Avoid When
- reasoning trace tokens count against context window — strip before storing in LanceDB or you pollute embeddings
- complexity heuristics can misfire on short but legally dense queries — add a manual override in the API request body
- some models leak reasoning tokens into the final answer if the delimiter is not strictly enforced — validate output format

## Reference
[hf:nvidia/NVIDIA-Nemotron-3-Ultra-550B-A55B-BF16](https://huggingface.co/nvidia/NVIDIA-Nemotron-3-Ultra-550B-A55B-BF16)
