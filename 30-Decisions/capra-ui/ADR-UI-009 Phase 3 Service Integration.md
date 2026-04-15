---
type: decision
status: stable
project: [capra-ui]
tags: [adr, capra-ui, integration, migration]
created: 2026-02-06
related: "[[ADR-UI-001 Adapter Pattern]], [[ADR-UI-004 Action Bus]], [[ADR-UI-008 Filter Registry]]"
---

# ADR-UI-009 Phase 3 — Service Integration Strategy

**Status:** Accepted
**Date:** 2026-02-06

## Context
App (`packages/app`) chamava HTTP direto via `fetch("/spr/query/execute")` em `schema.ts::executeQuery()`, ignorando o DataAdapter. Adapter só era usado para `applyFilter()`. Services do core (ActionBus, FilterManager, QueryManager) ficavam órfãos.

## Decision

1. **`executeRaw(mdx, options)` no DataAdapter** — dá ao app controle explícito sobre filtros (`ObjectFilterConfig`, `filterIds`, `noFilters`) que `fetchKpi`/`fetchList` não permitem.
2. **`useDataService` bridge no app, não no core** — lógica de `mergeFilters`, `GOVERNANCE_FILTERS`, `FILTER_IDS` é específica do BIMachine; poluiria a API genérica do core.
3. **Migração incremental por composable** — assinatura idêntica permite swap sem mudar chamadas. Casos especiais: `useCancelamentos`/`useFinanceiro` (executeQuery próprios), `useChart`/`useHeatmap` (fetch inline).
4. **ActionBus para reload-on-filter-change** — `useFilters` despacha `RELOAD_DATA` após `applyFilter`; App.vue registra handler. Polling de 500ms permanece como fallback para mudanças externas do BIMachine.
5. **Adiamentos:** FilterManager (incompatível com `ObjectFilterConfig`) e QueryManager caching (modelo skip/conflict incompatível com cache por hash).

## Consequences

### Positive
- Todo HTTP passa pelo adapter (testável, mockável)
- Reload imediato e desacoplado de filtros internos
- Sem breaking changes
- 16 testes para `executeRaw` + 13 para `useDataService`; 12 composables migrados

### Negative
- `mergeFilters`/`ObjectFilterConfig` permanecem em `schema.ts` (não migrados)
- Polling 500ms ainda necessário para mudanças externas
- FilterManager/QueryManager continuam não-usados pelo app
