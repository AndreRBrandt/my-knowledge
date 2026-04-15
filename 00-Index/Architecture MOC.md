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

## Backend (bode-api)
- `[[Strangler Fig Migration]]` — TS → Rust incremental
- `[[Auth Flow JWKS]]` — validação de token via JWKS (ES256)
- `[[Crate Boundaries]]` — regras de separação entre crates Rust
- `[[Multi-stage Docker Build]]` — build eficiente pra Rust
- `[[Blue-Green Deploy]]` — zero downtime
- `[[Two-pipeline Testing Strategy]]` — unit CI + integration homolog

## Observability
- `[[Structured Logging]]` — JSON condicional dev/prod
- `[[Request ID Propagation]]` — x-request-id end-to-end

## Frontend (capra-ui)
- _(a migrar)_ Adapter Pattern — abstração de fonte de dados
- _(a migrar)_ Action Bus — pub/sub entre componentes
- _(a migrar)_ Component Categories — analytic/layout/ui

## Cross-cutting
- _(a migrar)_ Hub Pattern — CLAUDE.md + docs/contexto/ (GADR-001)

---

Template: `[[Concept]]`
