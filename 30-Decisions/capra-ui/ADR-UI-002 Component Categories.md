---
type: decision
status: stable
project: [capra-ui]
tags: [adr, capra-ui, organization]
created: 2025-01-08
related: "[[Component Categories (UI vs Containers vs Analytics)]]"
---

# ADR-UI-002 Component Categories

**Status:** Accepted
**Date:** 2025-01-08

## Context
Crescimento do número de componentes exigia organização que facilitasse encontrar/entender propósito e manter consistência.

## Decision
Quatro categorias com hierarquia de dependência:

| Categoria | Propósito | Pode usar |
|-----------|-----------|-----------|
| `ui/` | Blocos básicos (BaseButton, Modal, Popover) | nada acima |
| `containers/` | Wrappers estruturais (AnalyticContainer) | `ui/` |
| `analytics/` | Exibem dados (KpiCard, DataTable) | `ui/`, `containers/` |
| `layout/` | Estrutura de página (AppShell) | qualquer |

Regras: 1 componente = 1 categoria. `ui/` nunca importa de `analytics/` ou `containers/`.

## Consequences

### Positive
- Hierarquia clara de dependências
- Documentação espelha estrutura (`docs/components/`)

### Negative
- Classificação ambígua em casos edge
- Refactor ao reclassificar

## References
- `[[Component Categories (UI vs Containers vs Analytics)]]`
- `[[ADR-UI-015 Dependency Boundaries]]`
