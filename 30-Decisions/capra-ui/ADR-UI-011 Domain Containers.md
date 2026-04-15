---
type: decision
status: stable
project: [capra-ui]
tags: [adr, capra-ui, containers, abstraction]
created: 2026-02-10
related: "[[Domain Containers]], [[ADR-UI-002 Component Categories]], [[ADR-UI-016 KPI Grid Layout]]"
---

# ADR-UI-011 Domain Containers

**Status:** Accepted
**Date:** 2026-02-10

## Context
`AnalyticContainer` é estrutural genérico (header, collapse, config popover, help modal, fullscreen, loading/error/empty states) — não conhece domínio. Toda lógica de domínio vivia nas pages: orquestração manual de KpiGrid + KpiCard + KpiCardWrapper + KpiConfigPanel + useKpiLayout + useKpiTheme + useDragReorder + 2 modais + maps de ícones/formatos/handlers. **~207 linhas de boilerplate por page**, com 4 maps duplicados, copy-paste entre pages.

## Decision
**Domain Containers** = camada acima do AnalyticContainer que encapsula toda a lógica de um tipo de objeto analítico. Arquitetura em 3 camadas:

1. **Page** — apenas dados + schema + handlers de negócio (~10-20 linhas)
2. **Domain Container** — `KpiContainer`, `DataTableContainer`, `ChartContainer` — encapsula grid, cards, config, DnD, modais, cores
3. **Primitivos** — `AnalyticContainer`, `KpiCard`, `KpiGrid`, etc. (sempre disponíveis para uso direto)

Schema é Single Source of Truth: ícone, formato, info modal, campos visíveis (cardFields, detailFields), categoria de cor.

## Consequences

### Positive
- ~90% menos código nas pages (~200 → ~15 linhas)
- Schema = SSoT (elimina KPI_ICONS, KPI_FORMAT, LAYOUT_ITEMS como maps separados)
- Consistência total (mesmo DnD/config/modais em todas pages)
- Mudanças na API do KpiCard afetam só o KpiContainer

### Negative
- Menos flexibilidade (mitigado por slots de escape `#card`, `#detail-modal`, `#config-extra`)
- Camada extra
- API surface grande no KpiContainer (mitigada por defaults)

## Alternatives Considered

| Alternativa | Por quê rejeitada |
|-------------|-------------------|
| `useKpiSection()` composable | Reduz script mas não template |
| Render function/headless | Perde legibilidade e debugging |
| Manter composição manual | Custo alto, duplicação inevitável |

## References
- `[[Domain Containers]]` — concept
