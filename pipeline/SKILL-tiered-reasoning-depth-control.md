# SKILL: Tiered Reasoning Depth Control

Use this pattern when: LLM inference costs spike unpredictably because all requests use maximum reasoning regardless of task complexity, making agentic pipelines expensive and slow for simple subtasks

**Category:** pipeline
**Confidence:** production-proven

## Core Logic
Attach a reasoning_level parameter (low|medium|high) to every LLM call. Route by task type: low=extraction/classification, medium=summarization/drafting, high=multi-step legal analysis/complex reasoning. In FastAPI, inspect the incoming request's task_type field and map it to reasoning_level before forwarding to the LLM. Example: if task_type in ['extract_clause','classify_doc']: reasoning_level='low' elif task_type in ['summarize','draft_section']: reasoning_level='medium' else: reasoning_level='high'. Cache high-reasoning outputs in LanceDB with their reasoning_level tag so identical complex queries reuse expensive results.

## When to Apply
- you have mixed-complexity tasks in one pipeline and want to control cost per request
- legal document processing has both cheap extraction and expensive reasoning steps
- agentic loops have many low-stakes intermediate steps that dont need full reasoning
- you want to expose a quality vs speed tradeoff knob to end users

## Avoid When
- low reasoning level may miss subtle legal nuances in clause extraction, always validate on legal corpus before routing real documents to low tier
- caching high-reasoning outputs can serve stale legal interpretations if underlying law changes, add TTL or invalidation hooks
- models without native reasoning_level param require prompt-engineering equivalents which are less reliable
- misclassifying task_type routes complex tasks to low tier silently producing wrong outputs with no error signal

## Reference
[hf:stepfun-ai/Step-3.7-Flash](https://huggingface.co/stepfun-ai/Step-3.7-Flash)
