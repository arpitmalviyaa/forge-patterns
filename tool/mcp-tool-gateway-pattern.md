# MCP Tool Gateway Pattern

**Category:** tool
**Confidence:** production-proven
**Observed in:** [firecrawl/firecrawl-mcp-server](https://github.com/firecrawl/firecrawl-mcp-server)
**Compatible with:** python, typescript, any, claude-code

## Problem
AI agents need to perform real-world actions (web scraping, search, page interaction) but have no standardized way to call external services, leading to bespoke brittle integrations per LLM client

## The Logic
Define a thin MCP server layer that wraps any external API (Firecrawl, Supabase, LanceDB, etc.) as named tools with typed input schemas. Agent calls tool by name -> MCP server validates input -> calls underlying API -> returns clean structured text/JSON back to agent context. Transport can be stdio (local), SSE, or HTTP Streamable. Each tool is a pure function: {name, description, inputSchema, handler}. Retry and rate-limit logic lives inside the handler, not the agent. Agent stays stateless about HOW the tool works.

## Steal This When
- Baby agent needs to call external APIs (scrape case law, search regulatory text, query Supabase) without baking API logic into the prompt or agent loop
- You want the same tool callable from Claude Desktop, Cursor, and your own FastAPI agent without rewriting integrations
- You need retry/rate-limit logic centralized away from the LLM orchestration layer
- Draft Terminal agent needs to autonomously fetch external legal sources or citations at runtime

## Gotchas
- stdio transport blocks concurrent tool calls — use SSE or HTTP Streamable for multi-agent or parallel workloads
- Tool descriptions ARE the API contract for the LLM — vague descriptions cause wrong tool selection, be precise and include example inputs in description
- MCP server process must stay alive for the duration of the agent session — add health checks if running in Docker/serverless
- Input schema validation errors surface as cryptic agent failures — always return structured error objects not raw exceptions
- npx cold-start latency (~2-3s) is fine for dev but use pre-installed global package or Docker image in production

## Real Implementation
https://github.com/firecrawl/firecrawl-mcp-server
