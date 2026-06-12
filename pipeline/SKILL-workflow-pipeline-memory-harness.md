# SKILL: Workflow Pipeline Memory Harness

Use this pattern when: Agent context windows bloat with raw conversation history, documents, and repeated facts causing high token costs and stale context; no structured way to persist, retrieve, and compress memories across sessions

**Category:** pipeline
**Confidence:** production-proven

## Core Logic
Define a staged workflow pipeline where each step declares explicit requires/produces keys: (1) ingest_resource: normalize raw input into canonical blob; (2) preprocess: modality-specific parsing to text/embedding-ready form; (3) extract_items: LLM extracts atomic typed facts (preferences, skills, relationships) into structured records; (4) dedupe_merge: idempotent upsert against existing memory store; (5) categorize_items: embed items, cluster into named categories with vector similarity, persist category->item edges; (6) persist_index: update category summaries lazily. On retrieval, run parallel RAG path (vector similarity on category->item->resource hierarchy) or LLM ranking path, apply where-scope filters, return only ranked relevant items. Each step is a small pure function with declared capability tags (llm, vector, db, io) allowing steps to be swapped, inserted, or removed without touching other steps. MemoryService acts as composition root wiring configs, backend, LLM profiles, and pipeline manager together.

## When to Apply
- You have an agent or chatbot that needs persistent memory across sessions without re-injecting full history
- Your LLM calls are expensive because context windows carry stale or redundant information
- You are building Draft Terminal and need to persist clause preferences, client tone profiles, or prior negotiation positions across documents
- You are building Greenlit and want to remember pitch feedback patterns, investor preferences, or prior conversation threads per user
- You need a portable memory layer that works across SQLite locally and Postgres in production without changing application code
- Your Baby agent needs to accumulate learned user preferences and task context between autonomous runs

## Avoid When
- Category bootstrap is lazy and scoped per user_id so first-run cold start may incur extra LLM calls to initialize embeddings; warm the category index on first ingest not on first retrieve
- The dedupe_merge step is currently a pass-through placeholder in this repo meaning duplicate memories accumulate over time; you must implement content-hash or semantic dedup before production use
- Scope field propagation via UserConfig.model merging into DB schema means changing user model fields late requires a migration across all four repository tables simultaneously
- Vector search falls back to brute-force cosine in SQLite; acceptable for small corpora but query time grows linearly so cap per-user item count or migrate to LanceDB/pgvector early
- LLM extraction quality determines memory quality entirely; a weak extraction prompt produces noisy categories that pollute retrieval ranking downstream
- Pipeline step requires/produces key contracts must be kept in sync manually; a mismatch silently passes None into downstream steps unless you add validation at registration time

## Reference
[NevaMind-AI/memU](https://github.com/NevaMind-AI/memU)
