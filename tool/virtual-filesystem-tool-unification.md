# Virtual Filesystem Tool Unification

**Category:** tool
**Confidence:** production-proven
**Observed in:** [strukto-ai/mirage](https://github.com/strukto-ai/mirage)
**Compatible with:** python, typescript, any

## Problem
Agent needs to interact with 5+ heterogeneous backends (S3, Postgres, Slack, GitHub, LanceDB) and accumulates N different SDK integrations, each with its own auth pattern, error model, and tool schema, making the tool manifest bloated and the agent context polluted with low-level API vocabulary

## The Logic
Define a single abstract Resource interface with POSIX-like operations (read, write, list, stat). Mount each backend at a path prefix under a shared Workspace root. Expose ONE set of shell-primitive tools (cat, ls, grep, cp, find) to the agent instead of backend-specific functions. Dispatch internally: when agent calls cat('/supabase/contracts/123.md'), router strips prefix, resolves to SupabaseResource, executes read, returns content. Agent never learns Supabase SDK. New backend = new Resource subclass + mount point, zero change to agent tool schema. Pattern: class Resource(ABC): async def read(path): ... async def write(path, data): ... async def list(path): ...; class Workspace: def mount(prefix, resource): ...; def execute(bash_cmd): parse -> dispatch -> resource.method()

## Steal This When
- agent needs to read/write more than 3 different backends and tool count is growing
- building Draft Terminal and agent needs to touch Supabase rows, LanceDB vectors, and local clause files in one reasoning chain
- Baby agent needs unified memory interface spanning RAM cache, Redis, and persistent Supabase without leaking storage details into prompt
- Greenlit agent must traverse TMDB data, user notes, and S3 assets in single pipeline without multiple SDK contexts
- you find yourself writing a new tool function every time you add a data source

## Gotchas
- POSIX semantics break down for append-only or schema-heavy backends like Postgres — you must define what 'ls /postgres/table' means and stick to it
- path-based dispatch requires careful prefix collision handling (e.g. /s3/reports vs /s3/reports-archive)
- async-native resource implementations required throughout — mixing sync SDK calls inside async dispatch causes event loop blocking
- LanceDB vector search has no clean POSIX analogue — you need a custom command override (like mirage's ws.command pattern) rather than forcing it into cat/grep
- snapshotting and workspace cloning only works if resources serialize their config not their state — easy to accidentally couple the two
- agent may still need backend-specific error vocabulary to surface meaningful failures — abstract too hard and you lose debuggability

## Real Implementation
https://github.com/strukto-ai/mirage
