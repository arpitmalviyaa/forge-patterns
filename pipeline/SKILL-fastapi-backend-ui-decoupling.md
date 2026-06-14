# SKILL: FastAPI Backend UI Decoupling

Use this pattern when: You need a production-grade Python API server with a custom frontend but are constrained to a platform or framework that imposes its own UI (Gradio, admin panels, etc.), or you want to serve a modern SPA alongside your FastAPI routes without spinning up a separate server

**Category:** pipeline
**Confidence:** production-proven

## Core Logic
Use the underlying FastAPI instance of a higher-level framework as your actual server. Register your own routes BEFORE the framework mounts its defaults so they take priority. Mount static SPA build artifacts under non-colliding path prefixes. Serve index.html as catch-all for SPA routing. Pattern: app = FrameworkServer(); attach_custom_routes(app); app.mount('/assets', StaticFiles(dir='dist/assets')); @app.get('/') -> FileResponse('dist/index.html'). The framework's backend utilities remain available but its UI is never shown.

## When to Apply
- You want FastAPI + Next.js/React SPA served from a single Python process without a reverse proxy
- You need to deploy to a platform that expects a specific framework entrypoint but you want full frontend control
- Draft Terminal needs to serve its Next.js build as static files from the same FastAPI process during local dev or constrained hosting
- You want to progressively replace a scaffolded UI with a custom one without rewriting the backend

## Avoid When
- Route registration ORDER is critical — custom routes must be attached before framework defaults or they will be shadowed
- StaticFiles mount paths must not collide with API route prefixes (e.g. do not mount /api as static)
- SPA catch-all route must come LAST or it will swallow API 404s
- Hot-reload in dev requires watching both the Python server and the frontend build output
- If the host framework updates its internal routes between versions your overrides may silently break

## Reference
[hf:build-small-hackathon/small-talk](https://huggingface.co/spaces/build-small-hackathon/small-talk)
