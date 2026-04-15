---
type: decision
status: stable
project: [capra-ui, capra-analytics]
tags: [adr, queries, mdx, conventions]
created: 2026-02-13
related: "[[ADR-UI-009 Phase 3 Service Integration]]"
---

# ADR-UI-014 App Query Building Conventions

**Status:** Accepted
**Date:** 2026-02-13

## Context
App constrói queries MDX para BIMachine com padrões recorrentes (ParallelPeriod, CrossJoin multi-dim, WHERE fixos vs dinâmicos). Sem convenções claras, cada composable reimplementava → duplicação e inconsistência.

## Decision

1. **Period helpers centralizados** em `usePeriodHelper.ts` (NUNCA copiar lógica de período localmente):
   - `buildKpiQueryWithPeriod`, `buildSimpleKpiQuery`, `buildTableQueryWithPeriod`
   - `getDateFilterInfo`, `extractValueAtIndex`, `createValueMapWithPeriod`

2. **CrossJoin queries**: helper `buildCrossJoinQueryWithPeriod` (descontos/queries.ts). Para CrossJoin constante (ex: 8 dims de detalhe), extrair como const e reutilizar.

3. **WHERE clauses**: filtros fixos (modalidade, turno) inline no MDX; dinâmicos (data, loja) via `objectConfig` do DataService; governança automáticos quando objectConfig ausente.

4. **SortDirection**: inline type literal `"ASC" | "DESC"` (2 valores não justificam type/enum).

5. **MDX result parsing** — formatos estáveis do BIMachine, parsers documentados:

| Formato | Parser | Locais |
|---------|--------|--------|
| `DD/MM/YYYY` | regex `/(\d{2})\/(\d{2})\/(\d{4})/` | usePeriodHelper, processors, useHeatmap, useFilterSync |
| `MMM/YYYY` | regex `/^([A-Z]{3})\/(\d{4})$/` | useChart |
| CrossJoin row | `.split(",")` | useProdutos, useVendedores |
| MDX member `[dim].[value]` | regex `/\[([^\]]+)\]/` | useProdutos, useVendedores |

## Consequences

### Positive
- Manutenção centralizada (mudança em período → 1 arquivo)
- Detecção de nível de data robusta (5 estratégias)
- CrossJoin constante evita divergência
- Doc clara dos pontos de update se formato mudar

### Negative
- Period helpers usam datasource Gold hardcoded — se novo datasource precisar de período, criar helper parametrizado
