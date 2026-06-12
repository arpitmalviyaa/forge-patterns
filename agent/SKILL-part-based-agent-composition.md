# SKILL: Part-Based Agent Composition

Use this pattern when: Agent logic becomes a monolithic blob: prompts, tools, memory, persona, and UI all tangled together making it impossible to swap components, test in isolation, or let users customize behavior without touching core code

**Category:** agent
**Confidence:** production-proven

## Core Logic
Define each agent capability as a self-contained 'part' (a directory with a standard entry point and a typed interface contract). A central loader discovers and hydrates parts at runtime by reading a known interface declaration. Parts declare what they provide (tools, persona, memory, world context) via a typed API contract (e.g. CharAPI_t). The runtime assembles a prompt_struct by calling each loaded part's contribution method, merging outputs into a unified structure before LLM dispatch. Hot-swap is possible because each part only depends on the shared interface, not on other parts. Pseudocode: loader.discover('./parts/**') -> for each part: module = dynamic_import(part/main); if implements(module, PartInterface): registry.register(part_type, module); on_request: prompt = PromptStruct(); for part in registry.get_all(): part.contribute(prompt, chat_metadata); llm_response = dispatch(prompt)

## When to Apply
- your agent needs swappable personas or domain-specific behaviors (e.g. Draft Terminal switching between clause-drafting mode and redline mode)
- you want users to install community-built tools/plugins without forking core agent code
- you are building Baby Agent and need to add new skills (web search, LanceDB retrieval, Supabase write) as drop-in modules without touching orchestration logic
- Greenlit needs per-project context injectors that activate only when relevant metadata is present
- Parkfields needs tenant-specific compliance rules injected as parts without a code deploy

## Avoid When
- Interface drift: if the shared PartInterface contract changes without versioning, all parts silently break — pin a schema version in every part manifest
- Security surface: dynamically loaded parts can execute arbitrary code; for multi-tenant SaaS (Parkfields, Draft Terminal) always sandbox or allowlist part sources rather than accepting community parts blindly
- Circular contribution: two parts that each read and write the same prompt_struct field create non-deterministic prompt assembly order — enforce a strict contribution phase ordering (world -> persona -> memory -> tools -> instructions)
- Dynamic import cold-start latency compounds with number of parts; cache hydrated modules across requests in a registry singleton rather than re-importing per request

## Reference
[steve02081504/fount](https://github.com/steve02081504/fount)
