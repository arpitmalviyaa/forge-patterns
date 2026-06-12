# SKILL: Semantic Context Layer Agent

Use this pattern when: AI agents querying your database return wrong or hallucinated results because they only see raw schemas with no business meaning, relationships, or domain vocabulary — the agent doesn't know that 'ARR' means annual recurring revenue or that 'active_cases' excludes archived records

**Category:** context
**Confidence:** production-proven

## Core Logic
1. Define a semantic model (MDL) that maps raw schema columns to business terms, adds descriptions, join paths, and example values. 2. Expose this model via a CLI/SDK that agents can query with natural language. 3. Agent calls `ask('<question>')` which retrieves relevant semantic context + generates validated SQL. 4. Dry-run validation catches bad SQL before execution. 5. All definitions are versionable (git-friendly) so context is reproducible across agent runs. Pattern: raw_schema + MDL_definitions + value_profiles + example_queries -> grounded_SQL. The key insight is separating 'what the data physically is' from 'what the data means to this business' into an explicit, queryable artifact that every agent shares.

## When to Apply
- your agent needs to query Supabase/Postgres and keeps generating wrong SQL because it misreads column names or join logic
- you have multiple agents or prompts all rediscovering the same business rules from scratch
- legal documents have domain-specific terminology (clause types, party roles, obligation states) that a raw DB schema cannot express
- you want reproducible, auditable query generation — e.g. Draft Terminal showing clause frequency analytics to users
- you are building a feature where a user asks a natural-language question and gets a table/chart back (GenBI pattern)

## Avoid When
- MDL definitions become a maintenance burden — if schema changes and MDL is not updated, agent gets stale context that may be worse than no context
- semantic layer adds latency: context retrieval + SQL generation + dry-run validation is 2-4 extra LLM/DB round trips
- value profiling (scanning column values to help the LLM) can be expensive on large Supabase tables — profile selectively
- agents can over-rely on the semantic layer and fail to handle edge cases not covered by MDL definitions
- the 'skills get onboarding' pattern assumes a long-running CLI session — in serverless FastAPI endpoints you need to cache the MDL context, not re-fetch it per request

## Reference
[Canner/WrenAI](https://github.com/Canner/WrenAI)
