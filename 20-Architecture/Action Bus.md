---
type: concept
status: stable
project: [capra-ui]
tags: [architecture, capra-ui, services, coordination, command-bus]
created: 2025-02-05
related: "[[ADR-UI-004 Action Bus]], [[Filter Registry]], [[Adapter Pattern (UI Layer)]]"
---

# Action Bus

> Command Queue centralizado que serializa, valida e debounce ações da UI antes de delegá-las a managers especializados.

## O que é
Implementação do padrão Command Bus. Componentes da UI **não chamam** managers diretamente; eles **despacham ações** via `actionBus.dispatch(action)`. O bus orquestra a execução.

## Por que existe
Sistema sem coordenação central tinha race conditions (`applyFilter` chamado N vezes simultaneamente sobrescrevia estado), queries disparadas antes de filtros aplicarem (não-await), zero debounce (cliques rápidos = N queries), conflitos detectados tarde (no execute, não no dispatch).

## Como funciona
1. Componente despacha ação (`{ type: 'APPLY_FILTERS', payload }`)
2. Bus aplica debounce (default 300ms)
3. Bus valida ação (conflitos, permissões)
4. Bus delega para manager especializado:
   - **FilterManager** — batch + validate + await
   - **QueryManager** — cache + dedup + retry
   - **StateManager** — loading/error/data
5. Manager executa, retorna resultado
6. Bus notifica componentes interessados

Tipos de ação: `APPLY_FILTERS`, `EXECUTE_QUERY`, `EXECUTE_QUERIES`, `RELOAD_PAGE`, `INVALIDATE_CACHE`.

## Relações
- `[[ADR-UI-004 Action Bus]]` — decisão original
- `[[ADR-UI-009 Phase 3 Service Integration]]` — como o app conectou (RELOAD_DATA handler)
- `[[Filter Registry]]` — usado pelo FilterManager
- Composable bridge: `useActionBus()`

## Exemplos concretos
- `useFilters` despacha `RELOAD_DATA` após cada `applyFilter` → App.vue reload sem polling
- Antes: 14 cliques em 1s = 14 queries; depois: debounce → 1 query
