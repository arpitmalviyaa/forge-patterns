# SKILL: Scoped Memory Project Tagging

Use this pattern when: Agent memory bleeds across unrelated contexts (work vs personal, client A vs client B, legal doc vs screenplay), causing irrelevant recall and context pollution in multi-domain indie apps

**Category:** memory
**Confidence:** production-proven

## Core Logic
Assign every memory write a 'space' or 'project' tag at ingest time. At recall time, filter by active space before semantic search. Pattern: write_memory(content, space_id=current_project) -> store(vector + metadata{space_id}). read_memory(query, space_id=current_project) -> hybrid_search(query, filter={space_id}) -> return ranked results. Space isolation is a first-class field, not an afterthought filter. Optionally allow cross-space recall with explicit override flag.

## When to Apply
- your agent serves multiple clients or projects from one memory store
- Baby agent needs to separate task memory from user preference memory
- Draft Terminal must not let one client's legal context leak into another's document suggestions
- Greenlit needs to isolate per-project script notes from general industry knowledge
- Parkfields needs tenant-level memory isolation without separate DBs

## Avoid When
- Space IDs must be set at write time — retrofitting isolation onto an untagged LanceDB/Supabase pgvector table is painful migration work
- Hybrid search (semantic + keyword) must pass space_id as a hard filter NOT a soft score modifier or isolation breaks under high similarity pressure
- User profile / preference memories often need cross-space read access — model these as a separate 'global' space with explicit merge logic at retrieval
- Space tag cardinality explosion if users create too many micro-spaces — add a space registry with validation

## Reference
[supermemoryai/supermemory](https://github.com/supermemoryai/supermemory)
