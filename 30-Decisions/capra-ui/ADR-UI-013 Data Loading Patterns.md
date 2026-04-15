---
type: decision
status: stable
project: [capra-ui]
tags: [adr, capra-ui, data-loading, partial-failure]
created: 2026-02-13
related: "[[Data Loading Patterns]], [[ADR-UI-007 Generic Composables]]"
---

# ADR-UI-013 Data Loading Patterns

**Status:** Accepted
**Date:** 2026-02-13

## Context
Múltiplos padrões evoluíram organicamente: composables de página com `Promise.all` (7-14 queries), framework `usePageDataLoader` (genérico com race protection), `useDataLoader`/`useModalDataLoader`. Problema central: `Promise.all` causa **partial failure total** — 1 falha de 14 zera todas as KPIs e o usuário perde dados parciais válidos.

## Decision
Tabela decisória explícita:

| Cenário | Padrão |
|---------|--------|
| Composable existente com estado complexo | `executeAllSettled` (do useDataService) |
| Nova página/feature | `usePageDataLoader` (race protection built-in) |
| Modal com load de item | `useModalDataLoader` |
| Operação async genérica | `useDataLoader` |
| Page orquestrando composables | `Promise.allSettled` |

`executeAllSettled` retorna `{ data: null, skipped: true }` para falhas; extractors (`extractValueAtIndex`, `createValueMapWithPeriod`) tratam skipped → 0.

## Consequences

### Positive
- 1/14 falha não zera as outras 13
- Compatível com extractors existentes
- Migração incremental (não reescreve composables)

### Negative
- Dois padrões coexistem (legado vs novo) — aceitável durante migração
- `allSettled` legado sem race protection automática (baixo risco)

## Alternatives Considered

| Alternativa | Por quê rejeitada |
|-------------|-------------------|
| Migrar tudo p/ usePageDataLoader | Composables com 13+ refs não justificam reescrita |
| `Promise.all` + try/catch individual | Verbose, error-prone |

## Notes
Stubs `useAnalyticData` e `useKpiData` (framework) DEVEM usar `useDataLoader` base quando implementados.

## References
- `[[Data Loading Patterns]]` — concept
