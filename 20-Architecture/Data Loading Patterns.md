---
type: concept
status: stable
project: [capra-ui, capra-analytics]
tags: [architecture, capra-ui, data-loading, partial-failure, race-protection]
created: 2026-02-13
related: "[[ADR-UI-013 Data Loading Patterns]], [[Generic Composables Layer]], [[Action Bus]]"
---

# Data Loading Patterns

> Tabela decisória de qual padrão de carregamento usar conforme o cenário, com `executeAllSettled` como pilar contra partial failure total.

## O que é
Cinco padrões que coexistem por design, cada um para um cenário específico:

| Cenário | Padrão | Race protection |
|---------|--------|-----------------|
| Composable existente, estado complexo (13+ refs) | `executeAllSettled` (do useDataService) | Manual |
| Nova página/feature | `usePageDataLoader` | Built-in |
| Modal com load de item | `useModalDataLoader` | Built-in |
| Operação async genérica | `useDataLoader` | Built-in |
| Page orquestrando composables | `Promise.allSettled` | N/A |

## Por que existe
`Promise.all` causava **partial failure total**: se 1 de 14 queries falhava, todas as KPIs zeravam. Usuário perdia 13 dados parciais válidos. `executeAllSettled` resolve sem reescrever composables — falhas retornam `{ data: null, skipped: true }` e extractors (`extractValueAtIndex`, `createValueMapWithPeriod`) tratam como 0.

## Como funciona

### `executeAllSettled` (composables existentes)
```typescript
const { executeAllSettled } = useDataService();
const [a, b, c] = await executeAllSettled([
  { query: queryA, options: { objectConfig: cfg } },
  { query: queryB, options: { objectConfig: cfg } },
  { query: queryC, options: { objectConfig: cfg } },
]);
// queries que falham → { data: null, skipped: true }
```

### `usePageDataLoader` (features novas)
- Race protection (cancela request anterior se filtros mudaram)
- Error recovery
- Refresh manual
- Recomendado para tudo novo

### `Promise.allSettled` (orquestração de composables)
```typescript
await Promise.allSettled([
  kpis.loadKpisData(),
  lojas.loadLojasData(),
  chart.loadChartData(),
])
// Cada composable gerencia seu loading/error
```

## Relações
- `[[ADR-UI-013 Data Loading Patterns]]` — decisão
- `[[Generic Composables Layer]]` — `useDataLoader` é base de todos
- `[[Action Bus]]` — orquestra reload-on-filter-change

## Migração
Coexistência intencional durante migração. Stubs `useAnalyticData` e `useKpiData` (framework) DEVEM usar `useDataLoader` quando implementados — sem race protection é débito técnico aceito apenas no legado.

## Princípio reutilizável
**Partial failure local > total failure**: APIs de orquestração devem isolar falhas individuais por default. `Promise.all` é um anti-pattern em UIs com múltiplas queries paralelas.
