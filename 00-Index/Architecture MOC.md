---
type: moc
status: living
created: 2026-04-15
---

# Architecture MOC

> Conhecimento **técnico** agnóstico de projeto. Padrões, estratégias, fluxos.

## Data Pipeline
- `[[Medallion Architecture]]` — padrão bronze/silver/gold
- `[[dbt Architecture]]` — pipeline Capra (raw → staging → intermediate → marts)

## Backend
- _(a migrar)_ Strangler Fig Migration — TS → Rust
- _(a migrar)_ Auth Flow JWKS — validação de token via JWKS
- _(a migrar)_ Multi-stage Docker Build — build eficiente pra Rust
- _(a migrar)_ Blue-Green Deploy — zero downtime

## Observability
- _(a migrar)_ Structured Logging — JSON condicional dev/prod
- _(a migrar)_ Request ID Propagation — x-request-id end-to-end

## Frontend
- _(a migrar)_ Adapter Pattern — abstração de fonte de dados (capra-ui)
- _(a migrar)_ Action Bus — pub/sub entre componentes (capra-ui)
- _(a migrar)_ Component Categories — analytic/layout/ui

## Cross-cutting
- _(a migrar)_ Hub Pattern — CLAUDE.md + docs/contexto/ (GADR-001)

---

Template: `[[Concept]]`
