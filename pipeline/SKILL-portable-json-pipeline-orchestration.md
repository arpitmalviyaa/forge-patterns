# SKILL: Portable JSON Pipeline Orchestration

Use this pattern when: AI workflows become tightly coupled to specific frameworks, hardcoded prompt chains, or vendor SDKs making them impossible to version-control, share, or redeploy without rewriting glue code

**Category:** pipeline
**Confidence:** production-proven

## Core Logic
Define every pipeline step as a node in a portable JSON schema: {id, type, config, inputs[], outputs[]}. Nodes declare their type (llm_call, vector_search, ocr, ner, transform) and connection topology separately from execution logic. A runtime loader reads the JSON, resolves the DAG, and dispatches each node to its registered handler. Result: swap LLM providers by changing one config key, add observability by wrapping the dispatcher, and version-control the entire workflow as a plain file. In Python: pipeline = load_json('workflow.json'); runner.execute(pipeline, inputs={}). In FastAPI: POST /run accepts pipeline JSON directly enabling dynamic user-defined workflows without code deploys.

## When to Apply
- you are building multi-step LLM workflows that need to evolve without redeployment
- users or power-users need to configure or customize AI pipelines (Draft Terminal clause templates, Greenlit scoring logic, Baby agent task chains)
- you want to swap between LLM providers or vector DBs (LanceDB vs Supabase pgvector) without changing application code
- you need to replay, debug, or log individual pipeline steps for observability
- pipeline definitions need to be stored, retrieved, and versioned in a database like Supabase

## Avoid When
- JSON schema versioning becomes critical debt when node types evolve — always include a schema_version field from day one
- DAG cycle detection must be explicit or circular pipelines silently hang or stack overflow
- Secrets and API keys must never be inlined in the portable JSON — use reference tokens resolved at runtime
- Dynamic user-defined pipelines introduce arbitrary code execution risk if custom node types are allowed — sandbox or whitelist node types strictly
- Observability hooks must be added to the dispatcher layer not individual nodes or you will miss cross-cutting latency data

## Reference
[rocketride-org/rocketride-server](https://github.com/rocketride-org/rocketride-server)
