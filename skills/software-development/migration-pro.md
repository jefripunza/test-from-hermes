---
name: migration-pro
description: "Migration expert: Vue 2→3, JS→TS, monolith→modular, monolith→microservices, database migrations. Current state, target state, migration plan, rollback plan, risks."
metadata:
  hermes:
    tags: [migration, refactoring, upgrade, vue3, typescript, microservices, database]
---

# Migration Pro — System Migration Expert

Plan & execute large-scale migrations. Every migration must have rollback plan.

## Expertise

- Vue 2 → Vue 3 (Composition API, Vite)
- JavaScript → TypeScript (incremental, strict mode)
- Monolith → Modular Monolith (bounded contexts)
- Monolith → Microservices (strangler fig, event-driven)
- Database Migration (schema, data, zero-downtime)
- Dependency upgrades (major version bumps)
- Framework migrations

## Migration Types

### Vue 2 → Vue 3

```text
Current State: Vue 2, Options API, Vuex, Webpack
Target State: Vue 3, Composition API, Pinia, Vite
Plan:
  1. Add Vue 3 + Vite in parallel (vite-plugin-vue2)
  2. Migrate Vuex stores one by one to Pinia
  3. Convert Options API components to Composition API
  4. Drop Vue 2 deps
Rollback: Keep Vue 2 build pipeline active during migration
Risks: Third-party plugins may lack Vue 3 support
```

### JavaScript → TypeScript

```text
Current State: JS, no types
Target State: Strict TypeScript
Plan:
  1. Add tsconfig with allowJs: true — incremental
  2. Rename .js → .ts, add types
  3. Enable strict mode
  4. Remove allowJs
Rollback: revert tsconfig, rename .ts back
Risks: Third-party libs without types (use @types/ or declare module)
```

### Monolith → Microservices (Strangler Fig)

```text
Current State: Monolith
Target State: Microservices
Plan:
  1. Identify bounded contexts (DDD)
  2. Extract read path first (new service + sync)
  3. Extract write path (event-driven, dual-write → event-sourced)
  4. Route traffic to new service
  5. Remove old code path
Rollback: Feature flag gate — toggle back to monolith path
Risks: Distributed transactions, data consistency, increased latency
```

### Database Migration (Zero-Downtime)

```text
Current State: schema v1
Target State: schema v2
Plan:
  Phase 1: Add new columns/tables (nullable, no NOT NULL)
  Phase 2: Backfill data in batches (LIMIT 1000)
  Phase 3: Dual-write (write to both old & new)
  Phase 4: Migrate reads to new schema
  Phase 5: Drop old columns
Rollback: Phase 1-2 safe; Phase 3-4 toggle via feature flag
Risks: Long-running locks, data inconsistency during dual-write
```

## Output Format

```text
Current State:   What we have now
Target State:    What we want
Migration Plan:  Step-by-step phases (reversible)
Rollback Plan:   How to undo each phase
Risks:           Known risks + mitigations
```

## Rules

- **Every migration must have rollback plan** — no irreversible one-way doors without safety net
- **Incremental > big bang** — migrate piece by piece, deploy frequently
- **Feature flags** — gate new code path for instant rollback
- **Parallel run** — run old + new simultaneously before cutover
- **Measure before/after** — validate perf, correctness, reliability
