# Tiered Reasoning Budget Agent

**Category:** agent
**Confidence:** production-proven
**Observed in:** [hf:stepfun-ai/Step-3.7-Flash](https://huggingface.co/stepfun-ai/Step-3.7-Flash)
**Compatible with:** python, typescript, any

## Problem
LLM inference costs spiral unpredictably when every agentic call uses maximum reasoning depth, or quality degrades when you cheaply route everything through a fast model with no depth control

## The Logic
Expose a reasoning_effort parameter (low|medium|high) on every LLM call site. Route requests through a decision function: classify the incoming task by complexity signal (simple lookup vs. multi-step legal analysis vs. code patch generation), then map that signal to a reasoning tier before dispatching. Fast/cheap tier handles classification, extraction, summarisation; medium tier handles multi-hop reasoning, tool chaining; high tier reserved for adversarial or high-stakes decisions. Cache aggressively at the prompt level to convert cache-miss tokens ($0.20/M) to cache-hit tokens ($0.04/M) — a 5x cost reduction for repeated system prompts and few-shot examples. Implementation sketch: def dispatch(task, context): effort = classify_effort(task) # returns 'low'|'medium'|'high' return llm_client.chat(model='step-3.7-flash', messages=build_messages(task, context), extra_body={'reasoning_effort': effort})

## Steal This When
- you have mixed task complexity in one agent pipeline and want to control cost without sacrificing quality on hard tasks
- legal document analysis (Draft Terminal) needs deep reasoning but clause extraction needs only shallow parsing
- Greenlit pitch scoring needs high-effort reasoning but metadata tagging needs low-effort
- Baby agent tool-call loops need medium reasoning for planning but low for tool result parsing

## Gotchas
- effort classification itself adds latency — keep the classifier ultra-cheap (rule-based or tiny model) or you negate savings
- prompt caching only activates when prefix tokens are identical — dynamic context injected before the cache boundary breaks cache hits; always append dynamic content at the end of the message array
- MoE models like this activate ~11B of 198B params per token so self-hosted cost math differs from API cost math — don't conflate benchmarks with your infra bill
- reasoning_effort field is provider-specific extra_body — abstract behind a wrapper so swapping providers doesn't require scattered code changes

## Real Implementation
https://huggingface.co/stepfun-ai/Step-3.7-Flash
