---
type: decision
status: stable
project: [capra-ui]
tags: [adr, capra-ui, dimensions, dynamic-discovery]
created: 2026-02-12
related: "[[Dimension Discovery]], [[ADR-UI-001 Adapter Pattern]]"
---

# ADR-UI-012 Dimension Discovery

**Status:** Accepted
**Date:** 2026-02-12

## Context
Membros de dimensões OLAP estavam hardcoded em schemas (`members: ["ALMOCO", "JANTAR"]`) e constantes (`TURNO_OPTIONS`, `MODALIDADE_OPTIONS`). Causou bugs como `[ITEM PADRÃO]` vs `[ITEM PADRAO]` (acento) e exigia deploy de código sempre que cubo recebia nova loja/turno/modalidade.

## Decision
Service `DimensionDiscovery` no `@capra-ui/core` que:

1. **Descobre membros via MDX `NON EMPTY`** no cubo (sem filtros do dashboard, via `executeRaw(mdx, { noFilters: true })`)
2. **Cache localStorage** com TTL configurável (default 1h)
3. **Background refresh** opcional
4. **Fallback gracioso** para `dimension.members` do schema se query falhar

Design: service puro (`DimensionDiscovery`) + composable bridge (`useDimensionDiscovery`) com provide/inject. Plugin provê automaticamente quando adapter presente. Mesmos patterns do QueryManager.

## Consequences

### Positive
- Elimina strings hardcoded divergindo do cubo real
- Novas lojas/turnos aparecem automaticamente
- Pattern reutilizável por qualquer app
- Cache eficiente (1x/hora)

### Negative
- Primeira carga ligeiramente mais lenta
- Dependência de localStorage
- Complexidade no init

## Alternatives Considered

| Alternativa | Por quê rejeitada |
|-------------|-------------------|
| Manter hardcoded | Causa de bugs |
| Discovery em build-time | Schemas sem acesso ao adapter |
| API dedicada de metadados | BIMachine não oferece — MDX é o caminho |

## References
- `[[Dimension Discovery]]` — concept
