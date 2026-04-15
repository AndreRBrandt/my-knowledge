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
- `[[Data Extraction Engine]]` — engine genérica em Rust (bode-sync) — connectors + queries metadata-driven
- `[[Teknisa BI Auth Flow]]` — auth Zeedhi (PHPSESSID + OAuth-Token + OAuth-Hash + OAuth-Project)

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
- `[[Adapter Pattern (UI Layer)]]` — tradução UI ↔ fonte de dados
- `[[Action Bus]]` — Command Queue + managers (Filter/Query/State)
- `[[Schema Builder]]` — fluent + registry + factories
- `[[Measure Engine]]` — calculators + transforms + formatters declarativos
- `[[Filter Registry]]` — multi-schema bindings
- `[[Theme System]]` — `[data-theme]` + tokens overrideable
- `[[Domain Containers]]` — 3-camada (page → container → primitivos)
- `[[Dimension Discovery]]` — descoberta dinâmica de membros via MDX
- `[[Component Categories (UI vs Containers vs Analytics)]]` — DAG de imports
- `[[Interaction System]]` — ações declarativas em componentes analíticos
- `[[Generic Composables Layer]]` — useAnalyticData, useDrillStack, useTableState
- `[[Data Loading Patterns]]` — partial failure local > total failure

## Cross-cutting
- `[[Framework-First Visual Ownership]]` — app não sobrescreve visual do framework
- `[[Core vs App Dependency Boundaries]]` — regra de import unidirecional + Teste do Segundo Consumidor

---

Template: `[[Concept]]`
