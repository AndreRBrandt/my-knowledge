---
type: moc
status: living
created: 2026-04-15
---

# Decisions MOC

> Todas as ADRs (Architecture Decision Records) do vault. Decisões que moldaram a arquitetura dos projetos.

## Global (GADRs — cross-project)

- `[[GADR-001 Strangler Fig TS to Rust Migration]]` — direção de migração workspace
- `[[GADR-002 Trunk-Based Development]]` — feature → main → tag → prod (squash merge)
- `[[GADR-003 Conventional Commits]]` — `type(scope): description` por projeto
- `[[GADR-004 Spec-Driven Workflow with Consultant LLM]]` — André lidera, LLM consultora
- `[[GADR-005 Framework-First UI Development]]` — app nunca sobrescreve framework
- `[[GADR-006 VPS as Stepping Stone Before Cloud]]` — DigitalOcean antes de cloud gerenciada
- `[[GADR-007 No SQL in Client]]` — toda query passa pela API server-side
- `[[GADR-008 BIMachine Deprecation]]` — substituição completa pelo stack próprio

## bode-api (Rust backend)

- `[[ADR-BODE-001 Rust Backend Language]]`
- `[[ADR-BODE-002 Axum sqlx serde Stack]]`
- `[[ADR-BODE-003 Docker Multi-stage Build]]`
- `[[ADR-BODE-004 Blue-Green Deployment]]`
- `[[ADR-BODE-005 Database Abstraction]]`
- `[[ADR-BODE-006 Three Environments]]`
- `[[ADR-BODE-007 CI CD Quality Gates]]`
- `[[ADR-BODE-008 GitHub Projects Management]]`
- `[[ADR-BODE-009 Zero Vendor Lock-in]]`
- `[[ADR-BODE-010 Observability Metrics]]` (deferred)

## capra-ui (Vue frontend framework)

- `[[ADR-UI-001 Adapter Pattern]]` — abstração de fonte de dados
- `[[ADR-UI-002 Component Categories]]` — UI / Containers / Analytics / Layout
- `[[ADR-UI-003 Interaction System]]` — ações declarativas (filter, modal, drawer, ...)
- `[[ADR-UI-004 Action Bus]]` — Command Queue centralizado
- `[[ADR-UI-005 Schema Builder]]` — fluent + registry + factories
- `[[ADR-UI-006 Measures and Transforms]]` — MeasureEngine declarativo
- `[[ADR-UI-007 Generic Composables]]` — useAnalyticData, useDrillStack, useTableState
- `[[ADR-UI-008 Filter Registry]]` — multi-schema bindings (proposed)
- `[[ADR-UI-009 Phase 3 Service Integration]]` — bridge useDataService + executeRaw
- `[[ADR-UI-010 Theme System]]` — `[data-theme]` + tokens
- `[[ADR-UI-011 Domain Containers]]` — KpiContainer / DataTableContainer / ChartContainer
- `[[ADR-UI-012 Dimension Discovery]]` — membros OLAP dinâmicos via MDX
- `[[ADR-UI-013 Data Loading Patterns]]` — executeAllSettled vs usePageDataLoader
- `[[ADR-UI-014 App Query Conventions]]` — period helpers + WHERE conventions
- `[[ADR-UI-015 Dependency Boundaries]]` — core ↔ app rules + Teste do Segundo Consumidor
- `[[ADR-UI-016 KPI Grid Layout]]` — `auto-fit` + `minmax` + token cascade
- `[[ADR-UI-017 Endpoint Schema Discovery]]` — plug-and-play schema (proposed)
- `[[ADR-UI-018 Design Token Enforcement]]` — zero hex hardcoded
- `[[ADR-UI-019 Framework First Visual Ownership]]` — design system contract

## capra-dbt

_(decisões implícitas no `[[dbt Architecture]]` e `[[Medallion Architecture]]` — sem ADRs formais ainda)_

## Superseded

_(vazio)_

---

## Sobre ADRs

Uma ADR captura: Context / Decision / Consequences / Alternatives.
Template: `[[Decision]]`

Naming:
- `ADR-BODE-NNN` — bode-api
- `ADR-UI-NNN` — capra-ui
- `GADR-NNN` — global cross-project
