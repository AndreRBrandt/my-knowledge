---
type: decision
status: stable
project: [capra-ui]
tags: [adr, capra-ui, composables]
created: 2025-02-05
related: "[[Generic Composables Layer]], [[ADR-UI-013 Data Loading Patterns]]"
---

# ADR-UI-007 Generic Composables

**Status:** Accepted (Fases 1-3 implementadas)
**Date:** 2025-02-05

## Context
Composables específicos de dashboard (useDescontos, useKpis, useLojas) tinham 2.000+ linhas cada, com lógica repetitiva (fetch, loading, modal, drill-down) misturada com regras de negócio. Framework deveria fornecer composables genéricos que encapsulam infraestrutura.

## Decision
Camada de composables genéricos no core:

| Composable | Responsabilidade |
|------------|------------------|
| `useAnalyticData` | Monta query + executa via QueryManager + processa via MeasureEngine + reativo a filtros |
| `useModalDrillDown` | Modal com load assíncrono de item |
| `useDrillStack` | Navegação em níveis (drill-down/up) |
| `useTableState` | Sort + filter + paginate + select |
| `useExport` | CSV / Excel / PDF |

Resultado: `useDescontos` foi de 2.847 → ~100 linhas, focado apenas em config + regras.

## Consequences

### Positive
- ~95% redução de código nas pages
- Separação infraestrutura vs negócio
- Bug fix em um lugar beneficia todos
- Consistência de comportamento

### Negative
- Trade-off entre convenção e customização
- Casos muito específicos podem não caber

## References
- `[[Generic Composables Layer]]` — concept
