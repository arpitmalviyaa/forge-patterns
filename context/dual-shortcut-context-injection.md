# Dual Shortcut Context Injection

**Category:** context
**Confidence:** production-proven
**Observed in:** [MatrixAges/polywise](https://github.com/MatrixAges/polywise)
**Compatible with:** python, typescript, any

## Problem
Agent sessions lose track of relevant files, prior knowledge, and available tools because context must be manually re-pasted each turn, bloating prompts and breaking flow for power users who switch between tasks

## The Logic
Implement two orthogonal injection namespaces at the chat input layer: (1) '@' resolves to saved artifacts — files, agents, wiki entries, memory nodes — and injects their content or a retrieval-compressed summary into the prompt context before the LLM call; (2) '/' resolves to registered tool/skill callables — search, fetch, summarise, run-code — and routes the message through that tool's execution pipeline instead of raw completion. Parser reads the first token of input: if '@' prefix, look up artifact registry, embed resolved content as system or user context block; if '/' prefix, route to skill dispatcher, execute, then optionally pipe result back into a follow-up completion. Both namespaces are composable in a single message: '@contract_template /summarise this' fetches the artifact AND runs the summarise skill on it. Registry entries are cheap — store id, type, short description, retrieval pointer. Actual content is lazy-loaded only when the shortcut fires.

## Steal This When
- users need to reference saved documents or prior knowledge mid-conversation without copy-paste
- you have a growing library of LanceDB or Supabase stored artifacts that need on-demand retrieval
- you want to expose discrete backend tools (search, fetch, clause-extract) to users without building a separate UI for each
- building Draft Terminal clause library lookup or Greenlit script context injection
- Baby agent needs to invoke a sub-skill from within a session without breaking conversation flow

## Gotchas
- '@' namespace must cap injected content length — large files will blow context window silently; always chunk or summarise before injection
- Skill dispatcher ('/') must validate tool names strictly; unrecognised slash commands should fail loudly not silently pass as plain text
- If both '@' and '/' appear together, order of resolution matters — resolve '@' first so skill receives hydrated context not a raw pointer
- Registry lookup adds latency before first token; use async pre-fetch triggered on '@' keystroke in UI if possible
- LanceDB vector search on '@' resolution needs a confidence threshold — low-score retrievals injected silently degrade answer quality without visible signal to user

## Real Implementation
https://github.com/MatrixAges/polywise
