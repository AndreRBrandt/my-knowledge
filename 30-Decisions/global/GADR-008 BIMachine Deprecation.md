---
type: decision
status: stable
scope: workspace
tags: [decision, global, workspace, deprecation, bimachine]
created: 2026-04-01
related: "[[GADR-001 Strangler Fig TS to Rust Migration]], [[GADR-007 No SQL in Client]], [[capra-dbt]]"
---

# GADR-008 BIMachine Deprecation

**Status:** Accepted (replacement in progress)
**Date:** 2026-04-01

## Context
BIMachine (plataforma BI proprietária) era backend OLAP original: cubos, MDX, ReduxStore, filtros via janela global. Limitações: vendor lock-in, latência, falta de visibilidade operacional, custo, sem controle sobre schema/mudanças, integrações através de iframe widget. Stack moderna substitui completamente: dbt (Postgres) + capra-api/bode-api + frontend Vue.

## Decision
**BIMachine é oficialmente obsoleto.** Não fazer integrações novas. Migrar dashboards existentes incrementalmente para o stack próprio:

- **Pipeline de dados:** dbt (`capra-dbt`) é source of truth — raw → staging → intermediate → marts em Postgres
- **API:** capra-api (Workers) hoje, bode-api (Rust) futuro
- **Frontend:** Vue + capra-ui consumindo `/api/v2/*` ou `/api/*`
- **Adapters:** `WorkersAdapter` substitui `BIMachineAdapter`; `MockAdapter` para testes

Documentação que mencionar BIMachine deve ser arquivada ou marcada como legacy.

## Consequences

### Positive
- Vendor lock-in eliminado
- Custo reduzido
- Controle total sobre schema, queries, performance
- Stack moderna, observável (Redis, dbt, Postgres, Rust)
- Sem dependência de iframe / widget integration

### Negative
- Migração tem custo de tempo (cada dashboard precisa rebuild)
- Conhecimento BIMachine acumulado torna-se débito a aposentar
- Algumas features OLAP-nativas (drill-through MDX) precisam ser reimplementadas

### Revisitar quando
- Nunca. Decisão é direção estratégica, não tática.

## References
- `[[GADR-001 Strangler Fig TS to Rust Migration]]` — backend coexiste durante migração
- `[[GADR-007 No SQL in Client]]` — disciplina herdada do redesign
- `[[capra-dbt]]` — pipeline canônico de dados
- `[[Adapter Pattern (UI Layer)]]` — mecanismo de swap
