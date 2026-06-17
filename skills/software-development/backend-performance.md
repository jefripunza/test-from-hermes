---
name: backend-performance
description: "Backend performance expert: pprof, benchmarking, CPU/memory profiling, GC analysis, Redis caching, connection pooling. Low latency, efficient resource usage."
metadata:
  hermes:
    tags: [backend, performance, profiling, pprof, caching, redis, go, optimization]
---

# Backend Performance — Low Latency Optimization

Low latency. Efficient resource usage. Profile first, optimize second.

## Expertise

- **pprof** — Go CPU, memory, goroutine, mutex, block profiling
- **Benchmarking** — `go test -bench`, benchstat comparison
- **CPU Profiling** — hotspot analysis, flame graphs, critical path
- **Memory Profiling** — allocation patterns, heap analysis, leak detection
- **GC Analysis** — GC tuning, pause times, allocation rate, escape analysis
- **Redis Caching** — cache-aside, write-through, distributed caching
- **Connection Pooling** — pool sizing, max open, max idle, lifetime
- **I/O Optimization** — buffered I/O, async, batch operations
- **Concurrency Optimization** — worker pools, rate limiting, backpressure

## Profiling (Go pprof)

```bash
# CPU profile
go test -bench=. -cpuprofile=cpu.out
go tool pprof -http=:8080 cpu.out

# Memory profile
go test -bench=. -memprofile=mem.out
go tool pprof -http=:8081 mem.out

# Runtime profile (live server)
import _ "net/http/pprof"
# GET /debug/pprof/profile (CPU, 30s)
# GET /debug/pprof/heap (memory)
# GET /debug/pprof/goroutine (goroutine dump)
```

### Flame Graph Interpretation

- **Wide bars** = expensive functions (most time/allocations)
- **Tall stacks** = deep call chains (potential simplification target)
- **Hotspots** — focus on top 3 widest paths first

## GC Analysis

```go
// GODEBUG=gctrace=1 ./app
gc 1 @0.002s 2%: 0.010+0.50+0.008 ms clock, 0.080+0.13/0.44/0.55+0.064 ms cpu
//   ^     ^   ^   ^-- STW sweep   ^-- assist/mark   ^-- STW mark termination
//   |     |   +-- CPU fraction
//   |     +-- time since start
//   +-- GC number
```

Goals:
- GC CPU < 5% of total
- Pause times < 1ms (p99)
- Allocation rate < 500MB/s

Tuning:
- `GOGC=100` (default) — tradeoff CPU vs memory
- `GOMEMLIMIT=8GiB` — soft memory limit (Go 1.19+)
- Reduce allocations: object pooling, preallocate slices, avoid escape to heap

## Caching Strategy

| Cache | Latency | Use Case | TTL |
|-------|---------|----------|-----|
| In-memory (sync.Map) | ~ns | Hot data, small cardinality | seconds |
| Redis | ~ms | Shared cache, sessions, rate limits | minutes |
| CDN | ~10ms | Static assets, public API | hours |
| HTTP cache | varies | API responses, idempotent GETs | depends |

```go
// Cache-aside with Redis
func getUser(ctx context.Context, id string) (*User, error) {
    key := "user:" + id
    data, err := redis.Get(ctx, key).Bytes()
    if err == nil {
        var u User
        json.Unmarshal(data, &u)
        return &u, nil
    }
    u, err := db.GetUser(ctx, id)
    if err != nil { return nil, err }
    if b, _ := json.Marshal(u); b != nil {
        redis.Set(ctx, key, b, 5*time.Minute)
    }
    return u, nil
}
```

## Connection Pooling

```go
// Database - optimal settings
db.SetMaxOpenConns(25)        // limit total connections
db.SetMaxIdleConns(10)        // keep idle connections ready
db.SetConnMaxLifetime(30*time.Minute)  // recycle connections
db.SetConnMaxIdleTime(5*time.Minute)   // close idle connections

// Rule of thumb: max_open = (cpu_cores * 2) + effective_spindles
```

## Benchmarking

```go
func BenchmarkParse(b *testing.B) {
    data := []byte("input data")
    for b.Loop() {
        Parse(data)
    }
}
```

```bash
go test -bench=. -benchmem ./...
go install golang.org/x/perf/cmd/benchstat@latest
benchstat old.txt new.txt  # Compare before/after
```

## Rules

- **Measure before optimizing** — profile first, find actual bottleneck
- **Explain bottlenecks** — what's slow, why, evidence from pprof
- **Explain tradeoffs** — memory vs speed, complexity vs gain, cache vs consistency

## Goal

- Low latency (p99 < 100ms for API, < 10ms for cache)
- Efficient resource usage (CPU < 70%, memory stable)
