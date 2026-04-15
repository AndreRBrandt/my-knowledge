---
type: hub
status: living
project: [bode-api]
tags: [rust, backend]
created: 2026-04-15
related: "[[bode-analytics-api]], [[dbt Architecture]]"
---

# bode-api

> Backend Rust do Capra. Substitui incrementalmente o `bode-analytics-api` (TypeScript/Hono) via Strangler Fig Migration.

## Stack
- **Language:** Rust (edition 2021, MSRV 1.88)
- **HTTP:** Axum 0.8 (tower middleware)
- **Database:** sqlx 0.8 (PostgreSQL, compile-time checked)
- **Async:** tokio
- **Serialization:** serde + serde_json
- **Auth:** jsonwebtoken (JWKS validation, ES256)
- **Error handling:** thiserror + anyhow
- **Observability:** tracing + tracing-subscriber (JSON em prod)
- **Infra:** Docker multi-stage, Traefik, VPS (DigitalOcean)

## Crates (workspace)
- `bode-core` — Domínio puro, ZERO I/O deps
- `bode-db` — Repositories, sqlx queries, pool
- `bode-server` — Axum routes, middleware, handlers
- `bode-sync` — ETL extractors + loaders

## Architecture
- `[[Strangler Fig Migration]]` — TS → Rust incremental
- `[[Auth Flow JWKS]]` — validação de token
- `[[Multi-stage Docker Build]]`
- `[[Blue-Green Deploy]]`
- `[[Structured Logging]]` — JSON condicional
- `[[Request ID Propagation]]` — x-request-id
- `[[Crate Boundaries]]` — regras de separação entre crates
- `[[Two-pipeline Testing Strategy]]` — unit CI + integration homolog

## Decisions (ADRs)
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

## Domain (RBAC)
O bode-api implementa o modelo RBAC da plataforma:
- _(a migrar)_ `[[RBAC Model]]` — roles, permissions, filiais
- _(a migrar)_ `[[AuthUser]]` — entidade do user autenticado

## External

- **Repo:** `github.com/bodedono/bode-api` (org bodedono)
- **Local:** `capra-workspace/bode-api/`
- **Homolog:** `bi-homolog.bodedono.com.br/api/v2/*`
- **Prod:** `bi.bodedono.com.br/api/v2/*` (strangler fig convive com TS em `/api/*`)

## Status atual

**Fase 1 — Fundação deploy:** ✅ Completa (PR #43 merged 2026-04-14)
- Dockerfile, docker-compose, CI com 6 quality gates, 49 testes, 92.54% coverage, JSON logging, request ID

**Fase 2 — Pipeline de deploy:** 🔄 Em andamento
- ✅ #41 Graceful shutdown (PR #44 merged 2026-04-15)
- 📋 **Plano expandido:** `[[bode-api Phase 2 Deploy Plan]]` (DRAFT — aguardando validação)
  - Story #35 — Deploy homolog (automático)
  - Story #36 — Deploy prod (blue-green)
  - Tasks abertas: #37 workflow homolog, #38 workflow prod, #39 script blue-green, #40 script rollback, #42 smoke tests
  - **Novos epics (modern practices):** VPS provisioning, Cosign signing, release-please, GH Environments + reviewer, Observability MVP

**Fase 3 — Features incrementais:** Pendente

## Active Work
- _(a migrar session logs)_

## Legacy being replaced

`[[bode-analytics-api]]` — TypeScript/Hono backend. Rotas `/api/*` migram incrementalmente pra `/api/v2/*` no Rust.
