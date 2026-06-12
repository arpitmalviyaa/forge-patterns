# Client-Token Namespace Isolation

**Category:** memory
**Confidence:** production-proven
**Observed in:** [hf:build-small-hackathon/her](https://huggingface.co/spaces/build-small-hackathon/her)
**Compatible with:** python, typescript, any

## Problem
Multi-tenant AI tools leak user data between sessions, require auth infrastructure, or expose private uploads in shared storage — a critical compliance risk in legal-tech where document confidentiality is non-negotiable

## The Logic
Generate a UUID in the browser via crypto.randomUUID(), store in localStorage, send as X-Client-Token header on every request. Server-side: ns = sha256(token)[:16], all storage lives under DATA_ROOT/<ns>/. No auth server, no user table. Retention sweep runs every N minutes deleting files older than TTL. Clear endpoint wipes entire ns dir. Result: each browser session is cryptographically isolated with zero backend auth complexity. Pseudocode: def ns_dir(token): return DATA_ROOT / sha256(token.encode())[:16] / def store_file(token, project, file): path = ns_dir(token) / sanitize(project) / file; path.parent.mkdir(); write(path). def sweep(): for ns_dir in DATA_ROOT.iterdir(): if ns_dir.name in PROTECTED: continue; for f in ns_dir.rglob('*'): if age(f) > RETENTION_HOURS: f.unlink()

## Steal This When
- users upload sensitive documents (contracts, legal filings) that must never be visible to other users
- you need multi-tenancy without a user auth system or database
- GDPR/retention compliance requires provable auto-deletion with no manual process
- Draft Terminal needs to store per-user document drafts or analysis results in Supabase/LanceDB without exposing them cross-user
- you want a clear data + no-telemetry guarantee you can show to legal clients

## Gotchas
- SHA256 of a UUID is only as secret as the UUID — if the token leaks from localStorage (XSS), the namespace is compromised; add Content-Security-Policy headers
- Sanitize project subdir names aggressively (re.sub non-alphanumeric to underscore) — path traversal via crafted project names can escape the namespace
- The sweep must explicitly protect shared dirs like _registry and _assets or it will delete your own infrastructure assets
- LanceDB table names derived from ns tokens need the same sanitization as filesystem paths — LanceDB rejects special chars silently or with opaque errors
- tab-close beacon (navigator.sendBeacon to /clear) is best-effort — browsers may not fire it; do not rely on it as the sole deletion path

## Real Implementation
https://huggingface.co/spaces/build-small-hackathon/her
