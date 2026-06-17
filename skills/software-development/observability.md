---
name: observability
description: "Observability expert: OpenTelemetry, Prometheus, Grafana, Sentry, structured logging, metrics, distributed tracing, alerting. Every critical flow observable, every error traceable."
metadata:
  hermes:
    tags: [observability, monitoring, opentelemetry, prometheus, grafana, sentry, logging, tracing]
---

# Observability — OpenTelemetry, Monitoring, Alerting

Every critical flow observable. Every error traceable. Every service exposes logs, metrics, and traces.

## Expertise

- **OpenTelemetry** — instrumentation, exporters, collectors, context propagation
- **Prometheus** — metrics exposition, recording rules, alerting rules, service discovery
- **Grafana** — dashboards, panels, annotations, alerting, Loki/Prometheus datasource
- **Sentry** — error tracking, performance monitoring, release tracking
- **Structured Logging** — JSON format, correlation IDs, log levels, sampling
- **Metrics** — RED (Rate, Errors, Duration) / USE (Utilization, Saturation, Errors), histograms, counters
- **Distributed Tracing** — trace context propagation, span attributes, sampling strategies
- **Alerting** — SLI/SLO-based alerts, on-call rotation, escalation policies

## Three Pillars

### Logs

```json
// Structured JSON logging (Go)
{"level":"info","time":"2024-01-01T12:00:00Z","msg":"request handled","method":"GET","path":"/api/users","status":200,"duration_ms":45,"trace_id":"abc123"}
```

```go
// Go — zerolog / zap
logger.Info().
    Str("method", r.Method).
    Str("path", r.URL.Path).
    Int("status", 200).
    Dur("duration", time.Since(start)).
    Msg("request handled")
```

### Metrics

```go
// Prometheus counters, histograms, gauges
var (
    httpRequestsTotal = prometheus.NewCounterVec(
        prometheus.CounterOpts{Name: "http_requests_total"},
        []string{"method", "path", "status"},
    )
    httpRequestDuration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name: "http_request_duration_seconds",
            Buckets: prometheus.DefBuckets,
        },
        []string{"method", "path"},
    )
)
```

### Traces

```go
// OpenTelemetry tracing
tracer := otel.Tracer("api-server")
ctx, span := tracer.Start(r.Context(), "handleRequest")
defer span.End()

span.SetAttributes(
    attribute.String("http.method", r.Method),
    attribute.String("http.path", r.URL.Path),
)
```

## Must Expose

| Service | Logs | Metrics | Traces |
|---------|------|---------|--------|
| HTTP API | ✓ | ✓ (RED) | ✓ |
| Worker | ✓ | ✓ (rate, processed) | ✓ |
| DB | ✓ (slow query log) | ✓ (connections, latency) | — |
| Cache | ✓ | ✓ (hit rate, latency) | — |
| Queue | ✓ | ✓ (depth, rate) | ✓ |

## Alerting (SLI/SLO)

```yaml
# PrometheusRule example
groups:
  - name: api-slos
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.01
        for: 5m
        labels: { severity: critical }
        annotations:
          summary: "API error rate > 1% for 5 minutes"
```

## Grafana Dashboard

Every service needs:
- Request rate (RPS)
- Error rate (%)
- Latency p50/p95/p99
- CPU/Memory usage
- GC pauses
- Open connections
- Queue depth (if applicable)
- Database query latency
- Cache hit ratio

## Goal

- Every critical flow is observable
- Every error is traceable
- Every service exposes logs, metrics, and traces
- Every alert has a runbook
