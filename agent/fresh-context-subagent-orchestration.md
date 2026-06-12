# Fresh Context Subagent Orchestration

**Category:** agent
**Confidence:** production-proven
**Observed in:** [open-gsd/gsd-core](https://github.com/open-gsd/gsd-core)
**Compatible with:** python, typescript, any, claude-code

## Problem
AI coding agent output quality silently degrades mid-task as context window fills with accumulated conversation, tool calls, and intermediate results — causing hallucinations, forgotten constraints, and inconsistent code generation on large features

## The Logic
Split every large AI task into a thin orchestrator + N fresh-context workers. Orchestrator holds only routing logic and reads persistent state from files (STATE.md, CONTEXT.md, config.json). Each worker (executor, verifier, planner) is spawned with a clean context window containing only the minimal artifact it needs. Pattern: 1) Orchestrator reads .planning/ state files to reconstruct minimal context, 2) Spawns subagent with focused prompt + relevant artifact slice (never full history), 3) Subagent writes result back to .planning/ file, 4) Orchestrator reads result, updates STATE.md, spawns next subagent. File system is the shared memory bus. No agent ever holds more than one phase worth of context.

## Steal This When
- building a multi-step AI pipeline where later steps degrade in quality (Draft Terminal clause generation chains)
- orchestrating Baby agent tasks that span more than 3-4 tool calls
- running parallel LanceDB retrieval + LLM synthesis where each branch needs isolated context
- any Greenlit workflow where research phase and write phase should not share token budget
- long Parkfields data extraction jobs that chain multiple LLM passes

## Gotchas
- Orchestrator must never accumulate subagent outputs in its own context — write to disk and re-read only what is needed next
- State files must be append-friendly and machine-parseable (structured Markdown or JSON) or subagents will misread partial state
- Absent-equals-enabled config defaults mean new feature flags silently activate — explicitly enumerate all flags at project init
- Parallel subagent waves require atomic per-task commits or a failed worker leaves state files in an inconsistent half-written state
- Fresh context means no implicit memory of earlier decisions — CONTEXT.md glossary must be injected into every subagent prompt or domain terms drift

## Real Implementation
https://github.com/open-gsd/gsd-core
