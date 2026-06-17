---
name: sre-lite
description: "SRE expert: reliability, availability, capacity planning, incident response, disaster recovery, error budgets. Production stability, fast recovery, predictable ops."
metadata:
  hermes:
    tags: [sre, reliability, availability, incident-response, disaster-recovery, error-budgets]
---

# SRE Lite — Site Reliability Engineering

Production stability. Fast recovery. Predictable operations. **Recovery strategy required.**

## Expertise

- **Reliability** — SLI, SLO, SLA design, error budgets
- **Availability** — uptime calculation, redundancy, HA architecture
- **Capacity Planning** — resource forecasting, load testing, auto-scaling
- **Incident Response** — severity levels, runbooks, on-call, postmortems
- **Disaster Recovery** — RTO, RPO, backup/restore, failover testing
- **Error Budgets** — SLO violation → stop shipping, focus on reliability
- **Chaos Engineering** — controlled failure testing

## SLI / SLO / SLA

```text
SLI (Service Level Indicator) — what we measure
  - Request latency p99 < 500ms
  - Error rate < 0.1%
  - Uptime >= 99.9%

SLO (Service Level Objective) — our target
  - 99.9% uptime per month
  - p99 latency < 500ms for 99% of requests

SLA (Service Level Agreement) — what we promise customers
  - 99.9% uptime guarantee
  - If breached: service credits
```

## Error Budget

```text
Error Budget = 100% - SLO
Example: 99.9% SLO → 0.1% error budget = ~43 min downtime/month

When budget exhausted: STOP shipping features, focus on reliability
When budget healthy: ship features, accept some risk
```

## Incident Response (Severity Levels)

| Severity | Definition | Response Time | Example |
|----------|------------|---------------|---------|
| SEV-1 | Complete outage, all users affected | <5 min | DB down |
| SEV-2 | Major feature degraded, many users | <15 min | API slow |
| SEV-3 | Minor issue, some users | <1 hour | UI bug |
| SEV-4 | Cosmetic, no users affected | next sprint | Typo |

## Runbook Template

```text
# Runbook: [Incident Name]

## Detection
How we know it's happening (alert, user report, dashboard)

## Triage
1. Is it a SEV-1? Escalate immediately
2. Check dashboard: [link]
3. Check logs: [link]

## Mitigation (immediate fix to stop bleeding)
Step-by-step commands

## Resolution (permanent fix)
Steps to fully resolve

## Verification
How to confirm fixed

## Postmortem
Document root cause, timeline, action items
```

## Disaster Recovery

```text
RTO (Recovery Time Objective): max acceptable downtime = 4 hours
RPO (Recovery Point Objective): max acceptable data loss = 1 hour

Strategy:
  - DB: hourly backups + WAL archive → point-in-time recovery
  - App: multi-region deployment with traffic routing
  - State: stateless app, externalized state (DB, object store)

DR Test: quarterly, automated failover
```

## Chaos Engineering

```text
Principles:
  1. Start with steady state — define normal behavior
  2. Hypothesize about failure — what should happen?
  3. Run experiment — kill pod, block network, saturate CPU
  4. Measure — did system behave as expected?
  5. Fix & repeat

Tools: chaos-mesh, litmus, gremlin
```

## Goal

- Production stability
- Fast recovery (SEV-1 <5min response)
- Predictable operations
- Recovery strategy required for every change
