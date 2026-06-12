# Hash-Anchored Idempotent Edit Tool

**Category:** tool
**Confidence:** production-proven
**Observed in:** [can1357/oh-my-pi](https://github.com/can1357/oh-my-pi)
**Compatible with:** python, typescript, any

## Problem
LLM-generated file edits fail silently or apply to wrong location when file has changed since read, causing corrupt diffs, wrong replacements, and wasted retry loops that burn tokens and break agent reliability

## The Logic
Before applying any file edit, compute a hash of the target content region (line range or string span) at read time and embed that hash as an anchor in the edit descriptor. At write time, re-hash the same region and compare before mutating. If hashes diverge, the tool returns a structured error with diff context rather than applying a corrupt patch. This makes edits idempotent across retries: same hash = same precondition = safe to apply again. Pseudocode: def apply_edit(path, target_text, replacement, anchor_hash): current = read_region(path, target_text); if hash(current) != anchor_hash: return EditError(stale=True, expected=anchor_hash, got=hash(current)); apply_replacement(path, target_text, replacement); return EditSuccess(). At read time: anchor = hash(region); return EditPayload(content=region, anchor=anchor). Model always echoes anchor back in edit call.

## Steal This When
- your agent edits files and you see wrong-location replacements in multi-turn sessions
- agent retries burning tokens because previous edit partially succeeded
- building a code-editing tool in Draft Terminal or Baby agent that needs reliable idempotent writes
- using LanceDB or Supabase to store file snapshots and need consistency guarantees between read and write tool calls

## Gotchas
- Hash must cover exact bytes the model was shown, including whitespace and line endings, or false positives on Windows CRLF vs LF
- If you summarize or truncate file content shown to model, hash the raw bytes not the summary or anchors will never match
- Hash granularity matters: too coarse (whole file) and concurrent edits to different regions block each other; too fine (single char) and any reformatter invalidates anchors
- Must store anchor in agent tool call state, not in the model context alone, or stateless FastAPI endpoints lose it between requests
- Retry loop must reread and recompute anchor before each retry, not reuse stale anchor from first attempt

## Real Implementation
https://github.com/can1357/oh-my-pi
