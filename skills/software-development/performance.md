---
name: performance
description: "Performance optimization expert: frontend (lazy loading, code splitting, bundle optimization) & backend (query optimization, profiling, concurrency, memory). Measure before optimize."
metadata:
  hermes:
    tags: [performance, optimization, profiling, frontend, backend, caching]
---

# Performance — Optimization Expert

Frontend + backend performance. **Measure before optimizing. Explain tradeoffs.**

## Frontend

### Lazy Loading

```typescript
// Route-level code splitting
const UserDashboard = () => import('./pages/UserDashboard.vue')

// Component-level
import { defineAsyncComponent } from 'vue'
const HeavyChart = defineAsyncComponent(() => import('./HeavyChart.vue'))
```

### Code Splitting
- Route-based splitting (default with Vite)
- Component-level splitting for heavy third-party
- Dynamic imports for modals, side panels
- Vendor chunk separation

### Bundle Optimization
- Analyze: `npx vite-bundle-analyzer`
- Tree-shaking — use named imports, avoid barrel files
- Image optimization — WebP/AVIF, responsive sizes, lazy loading
- Font subsetting — only needed glyphs
- Remove dead CSS — PurgeCSS via Tailwind

### Rendering Optimization
- `v-memo` for static lists
- `v-once` for static content
- `shallowRef` for large data (no deep reactivity)
- Virtual scrolling for long lists (`vue-virtual-scroller`)
- Debounce / throttle input handlers
- Avoid unnecessary watchers

### Asset Optimization
- Image: WebP/AVIF, srcset, lazy loading, CDN
- Font: woff2, subset, `font-display: swap`
- JS/CSS: minify, gzip/brotli, CDN cache

## Backend

### Query Optimization
- EXPLAIN ANALYZE — check for seq scans, large loops
- N+1 prevention — eager loading, batch queries
- Covering indexes — index-only scans
- Connection pooling — reuse, limit connections
- Pagination — cursor over offset for large datasets

### Profiling
```bash
# Go pprof
go test -bench=. -benchmem -cpuprofile=cpu.out -memprofile=mem.out
go tool pprof -http=:8080 cpu.out

# Python cProfile
python -m cProfile -o profile.out script.py
python -m pstats profile.out
```

### Memory Optimization
- Object pooling — reuse instead of allocate
- Slice preallocation — `make([]T, 0, expected)`
- Avoid large allocations in hot paths
- Stream processing instead of loading everything

### Concurrency Optimization
- Worker pools — bounded goroutines
- Rate limiting — prevent thundering herd
- Circuit breaker — fail fast, recover gracefully
- Bulkhead isolation — separate pools per subsystem

### Caching Strategy
| Cache | Use Case | TTL |
|-------|----------|-----|
| In-memory (sync.Map) | Hot data, low cardinality | seconds-minutes |
| Redis | Shared cache, sessions | minutes-hours |
| CDN | Static assets, public API | hours-days |
| HTTP cache | API responses | varies |

## Rules

- **Measure before optimizing** — profile first, actual bottleneck second
- **Explain bottlenecks** — what's slow, why it's slow, evidence
- **Explain tradeoffs** — speed vs memory, consistency vs latency, complexity vs benefit
