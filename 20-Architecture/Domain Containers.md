---
type: concept
status: stable
project: [capra-ui]
tags: [architecture, capra-ui, containers, three-layer]
created: 2026-02-10
related: "[[ADR-UI-011 Domain Containers]], [[Component Categories (UI vs Containers vs Analytics)]], [[Generic Composables Layer]]"
---

# Domain Containers

> Camada acima dos primitivos UI que encapsula a lógica completa de um tipo de objeto analítico (KPIs, tabelas, charts), eliminando ~200 linhas de boilerplate por page.

## O que é
Componentes especializados (`KpiContainer`, `DataTableContainer`, `ChartContainer`) que internamente compõem todos os primitivos necessários e expõem uma API declarativa baseada em schema + dados.

## Por que existe
`AnalyticContainer` é estrutural genérico (header, collapse, config popover, fullscreen, loading/error/empty). Não conhece domínio. Pages tinham que orquestrar manualmente: AnalyticContainer + KpiGrid + KpiCard + KpiCardWrapper + KpiConfigPanel + useKpiLayout + useKpiTheme + useDragReorder + 2 modais + 4 maps de ícones/formatos. **~207 linhas de boilerplate por page**, copy-paste entre pages, qualquer mudança afetava N pages.

## Como funciona

### Arquitetura em 3 camadas
```
Camada 3 — Page (app)            — 10-20 linhas, só dados + handlers
Camada 2 — Domain Container       — encapsula tudo (KpiContainer, etc.)
Camada 1 — Primitivos             — sempre disponíveis se uso direto for necessário
```

### Schema é Single Source of Truth
```typescript
interface KpiSchemaItem {
  key: string
  label: string
  icon: string                              // resolve por nome (Lucide)
  category: string                          // grupo de cores
  format: 'currency' | 'number' | 'percent'
  cardFields?: ('trend' | 'participation' | 'icon')[]
  detailFields?: ('previousValue' | 'variation' | ...)[]
  info?: { title, description, formula?, tips? }
}
```

### Page final fica trivial
```vue
<KpiContainer
  title="Indicadores"
  :kpis="computedKpis"
  :schema="KPI_SCHEMA"
  :default-visible="['faturamento', 'vendas']"
  storage-key="capra:vendas:kpis"
  collapsible
  @refresh="loadKpisData"
/>
```

### Slots de escape para customização
`#card`, `#detail-modal`, `#config-extra` — quando o caso edge precisa override pontual.

## Relações
- `[[ADR-UI-011 Domain Containers]]` — decisão
- `[[Component Categories (UI vs Containers vs Analytics)]]` — onde se encaixa (camada acima de containers genéricos)
- `[[Generic Composables Layer]]` — análogo no lado dos composables (mesma filosofia: encapsular padrões)

## Aplicação futura
Mesmo padrão para `DataTableContainer`, `ChartContainer`. Cada um usa AnalyticContainer como base estrutural.
