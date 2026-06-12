# SKILL: Filesystem-Native Agent Context

Use this pattern when: AI agents lack persistent, structured context about project constraints (brand rules, design tokens, domain vocabulary, workflow steps) so each invocation starts cold and produces inconsistent outputs

**Category:** agent
**Confidence:** production-proven

## Core Logic
Store agent-readable specification files (DESIGN.md, AGENTS.md, skills/*.md) directly in the project filesystem at well-known paths. Each file is a plain-text contract the agent reads before acting. Agents discover available skills/constraints by listing directories, not by calling APIs. Structure: /agents/AGENTS.md (repo map + cross-boundary rules), /skills/ (discrete callable units with metadata), /design-systems/ (domain vocabulary/constraints), /craft/ (universal rules any skill can opt into). Agent loop: 1) read AGENTS.md to orient, 2) read relevant skill/constraint file, 3) execute with constraints baked in, 4) write artifact back to filesystem. The filesystem IS the context window seed.

## When to Apply
- you have a multi-agent or repeat-invocation workflow where consistency across runs matters
- you want coding agents (Claude Code, Cursor, Copilot) to understand domain rules without prompt-stuffing
- you need a team-shareable brand or domain contract that agents and humans both read
- building Baby agent and want each sub-agent to self-orient from files not runtime injection
- Draft Terminal needs clause-style rules or legal vocabulary accessible to every agent invocation

## Avoid When
- agents must be explicitly instructed to read the context files first or they skip them
- file proliferation: too many skill files creates discovery overhead — keep a master index (AGENTS.md pattern)
- plain-text contracts go stale if not versioned alongside code — treat them as code, not docs
- filesystem approach breaks in pure serverless/stateless deployments (Topology C degradation in source repo)
- LanceDB or Supabase vector search should supplement not replace these files for large knowledge bases

## Reference
[nexu-io/open-design](https://github.com/nexu-io/open-design)
