---
type: concept
status: stable
project: [capra-ui]
tags: [architecture, capra-ui, dimensions, dynamic, cache]
created: 2026-02-12
related: "[[ADR-UI-012 Dimension Discovery]], [[Adapter Pattern (UI Layer)]]"
---

# Dimension Discovery

> Service que descobre dinamicamente os membros de uma dimensão OLAP via query MDX, com cache em localStorage e fallback gracioso para os membros estáticos do schema.

## O que é
- **`DimensionDiscovery`** — classe TS pura (sem Vue), faz query MDX `NON EMPTY` no cubo via `adapter.executeRaw(mdx, { noFilters: true })`
- **`useDimensionDiscovery`** — bridge composable com provide/inject
- Cache `localStorage` com TTL configurável (default 1h)
- Background refresh opcional via intervalo

## Por que existe
Schemas hardcodavam membros (`members: ["ALMOCO", "JANTAR"]`) e a app tinha constantes (`TURNO_OPTIONS`, `MODALIDADE_OPTIONS`) que divergiam do cubo real. Bug clássico: `[ITEM PADRÃO]` (com acento) vs `[ITEM PADRAO]` (sem). Adicionar nova loja/turno/modalidade exigia deploy.

## Como funciona

### MDX por dimensão
```mdx
SELECT {[Measures].[firstMeasure]} ON COLUMNS,
NON EMPTY {hierarchy} ON ROWS
FROM [dataSource]
```

`{ noFilters: true }` garante TODOS os membros (sem aplicar filtros do dashboard).

### Cache strategy
- Chave: `{storageKeyPrefix}:{schemaId}`
- TTL: configurável (default 3.600.000 ms = 1h)
- Storage: `localStorage` (persiste entre sessões)
- Invalidação manual: `invalidateCache()` / `clearCache()`

### Fallback
Se a query falha → usa `dimension.members` definido estaticamente no schema. App nunca trava por discovery.

## Relações
- `[[ADR-UI-012 Dimension Discovery]]` — decisão
- `[[Adapter Pattern (UI Layer)]]` — usa `executeRaw`
- `[[Schema Builder]]` — schema declara fallback estático
- Pattern paralelo: QueryManager + useQueryManager

## Padrão reutilizável
Qualquer app que use `@capra-ui/core` pode plugar: `provideDimensionDiscovery(adapter, options)`. Sem esforço por dashboard novo.
