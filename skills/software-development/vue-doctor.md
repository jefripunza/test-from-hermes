---
name: vue-doctor
description: "Vue application health doctor. Target: 100% Health Score — zero TS errors, zero ESLint errors, zero build errors, zero critical a11y issues."
metadata:
  hermes:
    tags: [vue, health, linting, type-check, audit, quality]
---

# Vue Doctor

Maintain healthy Vue application. Target: **Vue Doctor Health Score = 100%**.

## Required Stack

- Vue 3 + Composition API
- TypeScript (strict)
- Pinia
- Vue Router
- Vite

## Required Checks

```bash
npm run lint          # ESLint
npm run build         # Vite build
npm run test          # Vitest
npm run typecheck     # vue-tsc --noEmit
vue-tsc --noEmit      # Full type check
```

## Target Results

| Metric | Target |
|--------|--------|
| Vue Doctor Health | 100% |
| TypeScript Errors | 0 |
| ESLint Errors | 0 |
| Build Errors | 0 |
| Runtime Critical Errors | 0 |
| Accessibility Critical Issues | 0 |

## Review Areas

### Architecture
- Folder structure — clean separation of concerns
- Store organization — one concern per store
- Component organization — small focused components
- Composable organization — reusable extracted logic

### Type Safety
- No `any` — strict mode enforced
- No unsafe casts — `as` only with validation
- No missing types — all function signatures typed
- No implicit typing leaks — return types explicit

### Performance
- Unnecessary reactive state — avoid over-ref
- Unnecessary watchers — use computed where possible
- Over-rendering — check v-memo, key stability
- Missing lazy loading — route-level code splitting
- Bundle size issues — check Vite bundle analysis

### Maintainability
- Duplicate logic — extract composables/utils
- Duplicate components — create shared components
- Tight coupling — use props/emits, not parent access
- Dead code — remove unused imports, vars, components

## Output Format

Every Vue review must include:

```text
Health Score: X/100

Type Safety:
- issue / fix

Architecture:
- issue / fix

Performance:
- issue / fix

Accessibility:
- issue / fix

Maintainability:
- issue / fix

Required Fixes:
- fix
```
