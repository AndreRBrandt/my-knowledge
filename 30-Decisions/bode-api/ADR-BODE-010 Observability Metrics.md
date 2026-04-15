---
type: decision
status: stable
project: [bode-api]
tags: [adr, observability, metrics, deferred]
created: 2026-04-14
related: "[[bode-api]], [[Structured Logging]]"
---

# ADR-BODE-010 Observability & Performance Metrics

**Status:** Accepted (implementation deferred)
**Date:** 2026-04-14

## Context
We need visibility into application health, performance, and usage patterns — both at runtime (production traffic) and during deploy (build times, image sizes, deploy duration). Without metrics, we're flying blind: can't detect degradation, can't plan capacity, can't measure improvements.

## Decision
Implement a structured observability system covering two domains:

### Runtime Metrics (application in production)

| Metric | What it measures | Why |
|--------|-----------------|-----|
| Request count | Total requests per endpoint per status code | Traffic patterns, error rate |
| Response time | p50, p95, p99 latency per endpoint | Performance degradation detection |
| Active connections | Concurrent connections | Capacity planning |
| Data volume | Bytes in/out per endpoint | Bandwidth, payload optimization |
| Resource usage | CPU, memory, open file descriptors | Container limits, leak detection |
| DB query time | Latency per query type | Slow query detection |
| DB pool usage | Active/idle/waiting connections | Pool sizing |
| Auth metrics | Token validations, failures, latency | Security monitoring |
| Error rate | Errors by type (4xx, 5xx, panics) | Reliability |

### Deploy Metrics (CI/CD pipeline)

| Metric | What it measures | Why |
|--------|-----------------|-----|
| Build time | Cargo build duration (debug + release) | CI optimization |
| Image size | Docker image size per build | Bloat detection |
| Deploy duration | Time from tag to healthy container | Deploy reliability |
| Rollback count | How often we rollback | Stability indicator |
| Test duration | Time per test suite | CI speed |
| Coverage trend | Coverage % over time | Quality trend |

### Implementation approach (when ready)
- **Prometheus-compatible metrics** via `metrics` crate + `metrics-exporter-prometheus`
- **Endpoint:** `GET /metrics` (internal, not routed via Traefik)
- **Visualization:** Grafana (self-hosted on VPS or Grafana Cloud free tier)
- **Alerting:** Thresholds → Telegram (extends existing `resource-monitor.sh`)
- **Deploy metrics:** Emitted by CI workflow as structured logs, collected by same system

## Consequences

### Positive
- Data-driven decisions on performance and capacity
- Early detection of regressions
- Historical trends for planning
- Accountability: every deploy measured

### Negative
- Additional dependency (metrics crate, Prometheus/Grafana)
- VPS resource cost for Prometheus + Grafana (~200MB RAM)
- Maintenance burden for dashboards and alerts

## When to implement
After deploy pipeline is stable (Fase 2 complete). Metrics on a broken deploy pipeline have no value.

## Alternatives Considered

| Alternative | Why rejected |
|-------------|--------------|
| No observability | Can't measure improvements, detect degradation, or plan capacity |
| Only logs (no metrics) | Logs are events; metrics show trends. Need both. |
| Third-party SaaS (Datadog/New Relic) | Cost; self-hosted Prometheus/Grafana cheaper for our scale |

## References
- `[[bode-api]]` — project hub
- `[[Structured Logging]]` — complementary (logs ≠ metrics)
