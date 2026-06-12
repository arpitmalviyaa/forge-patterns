# Async Job Queue Polling

**Category:** pipeline
**Confidence:** production-proven
**Observed in:** [lfnovo/open-notebook](https://github.com/lfnovo/open-notebook)
**Compatible with:** python, typescript, any

## Problem
Long-running AI tasks (podcast generation, document processing, embeddings) block HTTP responses and cause timeouts or poor UX when handled synchronously

## The Logic
1. API endpoint receives request, submits job to queue (Surreal-Commands / any persistent queue), returns job_id immediately with 202 Accepted. 2. Background worker picks up job, executes expensive operation (LLM chain, TTS, embedding), writes status+result back to DB. 3. Client polls GET /commands/{job_id} on interval until status is 'complete' or 'failed'. 4. Frontend uses TanStack Query with refetchInterval that auto-stops when terminal state detected. Pattern: submit→ack→poll→consume. Queue backed by same DB already in stack (no extra infra). LangGraph workflow runs inside worker, checkpointed to SQLite so it survives worker restart.

## Steal This When
- user triggers AI operation taking >3 seconds (document ingestion, batch analysis, contract parsing, report generation)
- you want to avoid websockets but still give progress feedback
- running FastAPI without a separate Celery/Redis setup and want queue backed by existing DB (Supabase pg_boss, SurrealDB, or LanceDB metadata table)
- Baby agent task dispatch needs fire-and-forget with status tracking
- Draft Terminal clause extraction or Greenlit script analysis jobs need async processing without blocking the UI

## Gotchas
- Polling interval must back off exponentially or you hammer the DB — start at 1s, cap at 10s
- Job records must include created_at + TTL cleanup or the queue table grows unbounded
- If worker crashes mid-job, job stays 'running' forever — add heartbeat/timeout detection to reset stale jobs
- SQLite checkpoint storage for LangGraph is not safe for multi-worker deployments — swap to Postgres/Supabase for horizontal scale
- Frontend must handle the 'failed' terminal state explicitly and surface error message from job record, not just stop polling silently

## Real Implementation
https://github.com/lfnovo/open-notebook
