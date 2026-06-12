# Roots vs Features Route Split

**Category:** skill
**Confidence:** production-proven
**Observed in:** [lobehub/lobehub](https://github.com/lobehub/lobehub)
**Compatible with:** typescript, any

## Problem
SPA route files bloat with business logic, hooks, and UI components causing tangled navigation trees that are hard to test, refactor, or reuse across desktop/mobile/popup build targets

## The Logic
Separate route tree files (src/routes/) from business logic files (src/features/). Route files are THIN: only import from features and compose layout/page shell. Features are THICK: contain domain components, hooks, sidebar/header/body chunks, and expose clean index.ts exports. Register each route in ALL router config files simultaneously or a sync test will catch the mismatch. Pattern: route/[id]/index.tsx -> imports { PageComponent } from '@/features/Domain'; features/Domain/index.ts -> exports all domain UI and logic. Multiple build targets (web, mobile, desktop, popup) each have their own entry + router config but share the same features layer untouched.

## Steal This When
- Next.js or React Router SPA has routes that import hooks and business components directly causing merge conflicts when adding mobile or desktop variants
- Building a multi-surface app (web + desktop Electron + mobile) that needs to share feature logic but vary navigation structure
- Route files exceed ~30 lines because they contain inline components or data fetching logic
- Adding a second build target like a popup or CLI-embedded webview to an existing SPA

## Gotchas
- Forgetting to register new routes in ALL router config files causes silent blank screens on specific build targets only — add a sync test that diffs route paths across all config files to catch this automatically
- Feature index.ts barrel exports can cause circular imports if features import from each other — enforce a no-cross-feature-import lint rule or use a shared lib layer
- Thin route files that only re-export features can feel like useless indirection to new contributors — document the invariant explicitly in AGENTS.md or equivalent so it is not collapsed during refactors
- In Next.js App Router + SPA hybrid setups the route file convention collides with Next.js file conventions — use a distinct directory name like src/routes/ separate from src/app/ to avoid ambiguity

## Real Implementation
https://github.com/lobehub/lobehub
