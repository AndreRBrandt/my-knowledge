---
type: concept
status: stable
project: [capra-ui]
tags: [architecture, capra-ui, composables, vue, abstraction]
created: 2025-02-05
related: "[[ADR-UI-007 Generic Composables]], [[Measure Engine]], [[Data Loading Patterns]], [[Domain Containers]]"
---

# Generic Composables Layer

> Camada de composables Vue no `@capra-ui/core` que encapsula padrões de infraestrutura recorrentes (data fetching, drill-down, modal, table state, export), deixando ao app apenas configuração + regras de negócio.

## O que é
Conjunto de composables genéricos:
- `useAnalyticData` — query + cache + dedup + processamento (MeasureEngine) + reativo a filtros
- `useModalDrillDown` — modal com load assíncrono ao abrir
- `useDrillStack` — navegação em níveis (drill-down/up)
- `useTableState` — sort + filter + paginate + select
- `useExport` — CSV / Excel / PDF

## Por que existe
Composables específicos de dashboard tinham 2.000+ linhas (useDescontos, useKpis, useLojas), com lógica de infraestrutura (fetch, loading, modal, drill-down) misturada com regras de negócio. Mesmas funções reescritas em cada composable. Bug fix exigia mudar em N lugares.

## Como funciona

### Antes
```typescript
// useDescontos: 2.847 linhas, 13+ refs, modais inline, queries paralelas, processamento manual
```

### Depois
```typescript
export function useDescontos() {
  const lojas = useAnalyticData({
    schema: DescontosSchema,
    dimension: 'loja',
    measures: ['desconto', 'faturamento'],
    calculated: ['descontoVariacao', 'participacaoFaturamento'],
  })
  const drill = useDrillStack({
    levels: [{ id: 'lojas', dimension: 'loja' }, ...],
    onDrill: (level, item) => ({ [level]: item.name }),
    loadData: (level, filter) => loadDrillData(schema, level, filter),
  })
  const detalheModal = useModalDrillDown({ loadData: (item) => loadDetalheItem(item) })
  const tableState = useTableState({ data: lojas.data, defaultSort: { key: 'desconto', order: 'desc' } })
  return { lojas, drill, detalheModal, tableState, isLoading: computed(...) }
}
// ~100 linhas, foco 100% em config + regras
```

### Estrutura
```
src/core/composables/
├── data/    (useAnalyticData, useQueryBuilder, useDataTransform)
├── ui/      (useModalDrillDown, useDrillStack, useTableState, useSelection)
└── export/  (useExport)
```

## Relações
- `[[ADR-UI-007 Generic Composables]]` — decisão
- `[[Measure Engine]]` — `useAnalyticData` orquestra QueryManager + MeasureEngine
- `[[Data Loading Patterns]]` — quando usar cada composable de loading
- `[[Domain Containers]]` — análogo no lado de componentes (mesma filosofia)

## Trade-off
Casos muito específicos podem não caber → composição manual sempre disponível (composables continuam exportados como primitivos).
