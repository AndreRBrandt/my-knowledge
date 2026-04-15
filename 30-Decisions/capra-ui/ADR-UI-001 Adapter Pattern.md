---
type: decision
status: stable
project: [capra-ui]
tags: [adr, capra-ui, frontend, data-source]
created: 2025-01-06
related: "[[ADR-UI-008 Filter Registry]], [[Adapter Pattern (UI Layer)]]"
---

# ADR-UI-001 Adapter Pattern

**Status:** Accepted
**Date:** 2025-01-06

## Context
O framework precisa funcionar com diferentes fontes de dados (BIMachine em produção, Mock em dev/teste, futuras APIs REST/JSON). Componentes não devem conhecer detalhes de implementação da fonte.

## Decision
Interface `DataAdapter` em `src/adapters/types.ts` com métodos `fetchKpi`, `fetchList`, `getFilters`, `applyFilter`, `getProjectName`. Adapters concretos (`MockAdapter`, `BIMachineAdapter`) implementam o contrato. Detecção automática via `createAdapter()`.

## Consequences

### Positive
- Componentes desacoplados da fonte
- Testabilidade (MockAdapter), dev offline
- Fácil adicionar novos adapters

### Negative
- Camada extra de abstração
- Todos adapters precisam implementar interface completa

## References
- `[[Adapter Pattern (UI Layer)]]` — concept note
- `[[capra-ui]]` — project hub
