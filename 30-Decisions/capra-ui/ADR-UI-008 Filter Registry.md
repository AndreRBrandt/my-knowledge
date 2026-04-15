---
type: decision
status: living
project: [capra-ui]
tags: [adr, capra-ui, filters, multi-schema]
created: 2025-02-05
related: "[[Filter Registry]], [[ADR-UI-001 Adapter Pattern]], [[ADR-UI-006 Measures and Transforms]]"
---

# ADR-UI-008 Filter Registry (Multi-Schema)

**Status:** Proposed
**Date:** 2025-02-05

## Context
Dashboards consomem múltiplos schemas/cubos que podem: compartilhar filtros, ter filtros mapeados (Loja ↔ Filial ↔ Unidade), filtros exclusivos por schema, ou ignorar filtros específicos por query. No BIMachine: data é filtro global especial; cada cubo tem IDs próprios; queries declaram ignore lists.

## Decision
**FilterRegistry** define filtros lógicos (semânticos) e mapeia para filtros físicos por schema via `bindings`. **FilterManager** gerencia estado e sincroniza aplicação. Suporta:

- `valueTransform: (v) => transformedV` quando há padrão previsível
- `valueMap: { canonical: physicalValue }` quando não há padrão
- Ignore lists por query (`registerQuery(queryId, ignoreList)`)
- Filtros `global: true` (gerenciados pela plataforma host)

Estrutura: `src/core/filters/` (FilterRegistry, FilterManager, FilterSync).

## Consequences

### Positive
- Single source of truth de configuração de filtros
- Suporta múltiplos schemas com mapeamentos diferentes
- Type-safe, lógica isolada e testável

### Negative
- Setup inicial de bindings por schema
- Mais abstrações que sistema atual
- Risk: sincronização pode multiplicar chamadas API

## Alternatives Considered

| Alternativa | Por quê rejeitada |
|-------------|-------------------|
| Filtros independentes por schema | Duplica código e UI |
| Naming convention (mesmo nome = mesmo filtro) | Frágil |
| Filtro no adapter | Acopla demais |

## Notes
ADR-UI-009 deferiu adoção plena: modelo de bindings é incompatível com `ObjectFilterConfig` (skip on conflict, zeroOnConflict). Requer redesign antes de migrar app.

## References
- `[[Filter Registry]]` — concept
