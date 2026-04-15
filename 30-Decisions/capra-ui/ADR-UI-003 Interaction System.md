---
type: decision
status: stable
project: [capra-ui]
tags: [adr, capra-ui, interaction]
created: 2025-01-07
related: "[[Interaction System]], [[ADR-UI-004 Action Bus]]"
---

# ADR-UI-003 Interaction System

**Status:** Accepted
**Date:** 2025-01-07

## Context
Componentes analíticos (DataTable, KpiCard, Charts) precisavam responder a interações (filter, modal, drawer, navigate, emit) de forma padronizada. Cada componente implementava própria lógica → inconsistência.

## Decision
Composable `useInteraction` como camada central. Componente recebe `actions: ActionsConfig` via props, emite `InteractEvent` no click/dblclick/select, `useInteraction.handleInteract()` resolve a `ActionType` declarada.

```typescript
type InteractionType = 'click' | 'dblclick' | 'select' | 'hover'
type ActionType = 'filter' | 'modal' | 'drawer' | 'navigate' | 'emit' | 'custom'
```

## Consequences

### Positive
- Comportamento consistente entre componentes
- Configuração declarativa via props
- Fácil adicionar novos action types

### Negative
- Curva de aprendizado inicial
- Overhead para interações simples

## References
- `[[Interaction System]]` — concept
