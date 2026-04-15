---
type: decision
status: stable
project: [capra-ui]
tags: [adr, capra-ui, services, coordination]
created: 2025-02-05
related: "[[Action Bus]], [[ADR-UI-008 Filter Registry]], [[ADR-UI-009 Phase 3 Service Integration]]"
---

# ADR-UI-004 Action Bus

**Status:** Accepted (implementado 2025-02-05)
**Date:** 2025-02-05

## Context
Sistema sofria de race conditions em `applyFilter()`, queries disparadas antes dos filtros aplicarem, sem debounce, cache existia mas não era usado, conflitos detectados tarde demais.

## Decision
Padrão Command Bus / Action Queue com três managers especializados — FilterManager, QueryManager, StateManager — orquestrados pelo ActionBus que aplica debounce automático (default 300ms), validação preventiva, priorização e cancelamento de ações obsoletas.

Tipos de ação: `APPLY_FILTERS`, `EXECUTE_QUERY`, `EXECUTE_QUERIES`, `RELOAD_PAGE`, `INVALIDATE_CACHE`.

Implementação em `src/core/services/`.

## Consequences

### Positive
- Zero race conditions (ações serializadas por design)
- Debounce automático
- Cache integrado (queries deduplicadas)
- Conflitos detectados antes de executar
- Log centralizado de ações (observabilidade)

### Negative
- Curva de aprendizado
- Camada extra para ações simples
- Migração: código existente precisa adaptar

## References
- `[[Action Bus]]` — concept
- `[[ADR-UI-009 Phase 3 Service Integration]]` — como o app conectou
