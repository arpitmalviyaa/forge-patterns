# SSE Streaming JSON Frames

**Category:** pipeline
**Confidence:** observed
**Observed in:** [hf:huggingface-projects/diffusiongemma-codegen](https://huggingface.co/spaces/huggingface-projects/diffusiongemma-codegen)
**Compatible with:** python, typescript

## Problem
Long-running AI generation tasks (LLM inference, document analysis, clause extraction) block the HTTP response until completion, giving users no feedback and risking timeout on slow operations

## The Logic
Mount a dedicated streaming endpoint on your FastAPI app that yields newline-delimited JSON frames as the work progresses. A background thread or async generator pushes intermediate state into a queue; the endpoint drains that queue via SSE or chunked transfer. Frontend JS reads the stream with fetch+ReadableStream and renders each frame incrementally. Pattern: POST /generate -> StreamingResponse(generator) where generator yields '{"step": n, "partial": "...", "done": false}\n' until final '{"done": true, "result": "..."}'

## Steal This When
- Draft Terminal is streaming clause-by-clause redline output and you want the editor to populate token-by-token instead of waiting for full LLM response
- Baby agent has multi-step tool chains and you want the UI to show each tool call result as it completes rather than buffering all steps
- Any generation step exceeds 3 seconds and you need perceived responsiveness for the user

## Gotchas
- FastAPI StreamingResponse with a thread-fed queue requires careful thread lifecycle management — the generator must catch GeneratorExit to cleanly terminate the background thread when client disconnects mid-stream
- Supabase Realtime or Next.js fetch both need explicit header 'Content-Type: text/event-stream' and 'Cache-Control: no-cache' or the browser will buffer the entire response before rendering
- If your LanceDB retrieval step runs before streaming begins, ensure it completes synchronously before yielding the first frame or the stream will open then stall, which some proxies interpret as a timeout
- CPU/GPU boundary issue from the source: any state passed into the streaming generator must be serializable — avoid passing torch tensors or ORM model instances; convert to plain dicts/strings first

## Real Implementation
https://huggingface.co/spaces/huggingface-projects/diffusiongemma-codegen
