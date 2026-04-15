---
type: concept
status: living
project: [capra-ui]
tags: [architecture, capra-ui, filters, multi-schema, mapping]
created: 2025-02-05
related: "[[ADR-UI-008 Filter Registry]], [[Action Bus]], [[Adapter Pattern (UI Layer)]]"
---

# Filter Registry

> Camada que define filtros lógicos (semânticos) do dashboard e mapeia para filtros físicos de cada schema (cubo) — incluindo IDs, dimensões e transformações de valor.

## O que é
Duas peças:
- **FilterRegistry** — registro estático: define filtros lógicos (`loja`, `período`, `vendedor`) e seus `bindings` por schema (`adapterId`, `dimension`, `valueTransform`/`valueMap`).
- **FilterManager** — runtime stateful: aplica filtros sincronizadamente em todos schemas relevantes, gerencia ignore lists por query, expõe `state` reativo.

## Por que existe
Dashboards consomem múltiplos cubos onde o "mesmo filtro" tem nomes/IDs diferentes (Loja ↔ Filial ↔ Unidade), valores diferentes (`bdn_bv` ↔ `BDN_BV` ↔ `Bode do Nô - Boa Viagem`), e algumas queries devem ignorar filtros específicos (drill-down geral). Sem registry, cada composable replica essa lógica.

## Como funciona

### Bindings declaram mapping por schema
```typescript
loja: {
  type: 'multiselect',
  bindings: {
    vendas:     { adapterId: 1001, dimension: '[Loja]' },
    auditoria:  { adapterId: 2001, dimension: '[Unidade]' },
    financeiro: { adapterId: 3001, dimension: '[Filial]' },
  }
}
```

### Transformação de valor (canonical → físico)
- `valueTransform: (v) => v.toUpperCase()` quando há padrão previsível
- `valueMap: { 'bdn_bv': 'BDN(BV)' }` quando não há padrão

### Ignore lists por query
```typescript
filterManager.registerQuery('grafico-tendencia', ['loja']) // ignora loja
const filtros = filterManager.getFiltersForQuery('grafico-tendencia')
```

### Filtro `global: true`
Gerenciado pela plataforma host (ex: filtro de data no BIMachine).

## Relações
- `[[ADR-UI-008 Filter Registry]]` — decisão (status: proposed, adoção plena adiada)
- `[[Adapter Pattern (UI Layer)]]` — Manager usa adapter para `applyFilter`
- `[[Action Bus]]` — applyFilter despacha pelo bus

## Status
Adoção plena adiada (ADR-UI-009): modelo de bindings é incompatível com `ObjectFilterConfig` atual (skip on conflict, zeroOnConflict). Requer redesign antes de migrar app inteira.
