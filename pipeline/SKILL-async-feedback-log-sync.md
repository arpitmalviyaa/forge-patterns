# SKILL: Async Feedback Log Sync

Use this pattern when: Background threads for telemetry/feedback logging break under forked process environments (ZeroGPU, gunicorn prefork, FastAPI startup), causing file descriptor errors or silent data loss

**Category:** pipeline
**Confidence:** production-proven

## Core Logic
Append feedback records locally to a JSONL file first (guaranteed fast, never fails the main request), then synchronously push the file to remote storage from the main process. Avoid background threads or CommitScheduler-style daemons that don't survive forks. Pattern: write_local(record) -> upload_remote(file) where upload failure is caught and logged but never raises. Add UTC timestamp and UUID session ID to every record at log time, not at upload time.

## When to Apply
- You need lightweight telemetry/feedback logging in a FastAPI app running under gunicorn with multiple workers
- You want to log LLM interactions (prompt, response, latency, user rating) to Supabase or S3 without blocking the response
- Baby agent needs to log tool-call traces for later fine-tuning or debugging without crashing on worker respawn
- Draft Terminal needs to capture user edits and rejections of AI-generated legal clauses for feedback loop

## Avoid When
- Synchronous upload on every request adds latency; batch or debounce uploads if request volume is high
- JSONL file grows unbounded per process instance; add rotation or size cap to avoid disk fill
- Multiple gunicorn workers each write their own UUID-named JSONL file, so remote storage accumulates many small files; use a merge job or switch to append-only Supabase insert instead
- If HF_TOKEN / Supabase key is missing at startup, remote sync silently no-ops; add a startup health check asserting the credential is present
- UUID per session means replays or retries create duplicate files; use request_id passed from caller instead

## Reference
[hf:AlexWortega/my_pi_agent](https://huggingface.co/spaces/AlexWortega/my_pi_agent)
