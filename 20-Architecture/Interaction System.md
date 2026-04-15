---
type: concept
status: stable
project: [capra-ui]
tags: [architecture, capra-ui, interaction, declarative]
created: 2025-01-07
related: "[[ADR-UI-003 Interaction System]], [[Action Bus]]"
---

# Interaction System

> Camada declarativa que padroniza como componentes analíticos respondem a interações do usuário (click, dblclick, select, hover) → ação (filter, modal, drawer, navigate, emit, custom).

## O que é
Composable `useInteraction` + tipos `InteractionType`/`ActionType` + interface `ActionsConfig` que componente recebe via prop.

## Por que existe
Cada componente analítico (DataTable, KpiCard, Charts) implementava sua própria lógica de "o que fazer ao clicar". Inconsistência: KpiCard abria modal, DataTable disparava filtro, Charts não respondiam. Configurar interação exigia mexer no código de cada componente.

## Como funciona

### Tipos
```typescript
type InteractionType = 'click' | 'dblclick' | 'select' | 'hover'
type ActionType = 'filter' | 'modal' | 'drawer' | 'navigate' | 'emit' | 'custom'

interface ActionsConfig {
  click?: ActionConfig
  dblclick?: ActionConfig
  select?: ActionConfig
}
```

### Fluxo
1. Componente recebe `actions: ActionsConfig` via props (declarativo, configurável fora)
2. Usuário interage
3. Componente emite `InteractEvent` via `emit('interact', { type, payload, source })`
4. `useInteraction.handleInteract(event)` resolve ação correspondente
5. Executa: aplica filtro, abre modal/drawer, navega, emite custom event

### Vantagem do declarativo
Page configura `actions={{ click: { type: 'filter', target: 'loja' }, dblclick: { type: 'modal', component: 'LojaDetail' } }}` — comportamento mudável sem tocar componente.

## Relações
- `[[ADR-UI-003 Interaction System]]` — decisão
- `[[Action Bus]]` — ações que mexem em estado global (filter, query) podem despachar via bus
- `[[Domain Containers]]` — encapsulam configuração de interação típica por tipo de objeto

## Trade-off
Overhead para interações triviais (botão simples não precisa). Vale para componentes analíticos com múltiplos triggers.
