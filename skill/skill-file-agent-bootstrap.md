# Skill File Agent Bootstrap

**Category:** skill
**Confidence:** production-proven
**Observed in:** [iOfficeAI/OfficeCLI](https://github.com/iOfficeAI/OfficeCLI)
**Compatible with:** python, any, claude-code

## Problem
AI agents don't know how to use a new tool or API without lengthy system prompt engineering or manual configuration — onboarding friction kills automation pipelines

## The Logic
Create a SKILL.md file co-located with or hosted alongside your tool that encodes: (1) install instructions, (2) strategy heuristics (e.g. L1 read → L2 DOM edit → L3 raw XML), (3) help system entry points, (4) quick-start command examples, (5) performance modes, (6) schema discovery commands. Agent reads this single file and becomes immediately capable. The file is both human-readable docs AND machine-executable agent instructions — same artifact serves both audiences. Pattern: `curl https://yourtool.ai/SKILL.md` teaches any agent everything it needs. For your stack: expose a /skill endpoint on your Flask/FastAPI service returning a markdown skill file, or commit SKILL.md to repo root. Agent loads it once via tool call or curl, then operates autonomously.

## Steal This When
- you are building a CLI tool or API that AI agents will call repeatedly
- you want Claude Code or Cursor to use your Draft Terminal or Greenlit API without a massive system prompt
- you need agents to self-bootstrap tool knowledge in Baby agent workflows
- you want to encode strategy heuristics (prefer high-level over low-level operations) so agents don't write 50-line Python when one API call suffices
- you want the same doc to serve human devs and AI agents

## Gotchas
- Skill file must encode failure recovery paths — if agent can't find the binary it needs a fallback install sequence baked in, not just the happy path
- Skill files go stale if not versioned alongside the tool — pin a version header so agents can detect mismatch
- If skill file is too long agents may truncate or lose context — use layered disclosure: overview first, verbose schema discoverable via help commands rather than inlined
- Agents may cache skill content across sessions — include a cache-bust hint or version field
- For LanceDB/Supabase tool integrations, the skill file pattern works best when paired with a structured JSON schema endpoint so agents can programmatically introspect available operations

## Real Implementation
https://github.com/iOfficeAI/OfficeCLI
