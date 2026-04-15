---
type: moc
status: living
created: 2026-04-15
---

# Decisions MOC

> Todas as ADRs (Architecture Decision Records) do vault. Decisões que moldaram a arquitetura dos projetos.

## Global (GADRs — cross-project)

_(a migrar — 7 GADRs de `bi_projects/docs/contexto/DECISOES_GLOBAIS.md`)_

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

_(a migrar — ~15 ADRs ativas, pular 005/007/009/017 obsoletos)_

## capra-dbt

_(a migrar — decisões sobre pipeline de dados)_

## Superseded

_(vazio — ADRs substituídas ficam aqui quando aplicável)_

---

## Sobre ADRs

Uma ADR captura:
1. **Context** — situação e restrições
2. **Decision** — o que foi decidido
3. **Consequences** — positivo, negativo, riscos
4. **Alternatives Considered** — opções descartadas e por quê

Template: `[[Decision]]`

Naming:
- `ADR-BODE-NNN` — decisões do bode-api
- `ADR-UI-NNN` — decisões do capra-ui
- `GADR-NNN` — decisões globais cross-project
