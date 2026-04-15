---
type: concept
status: stable
project: [capra-ui]
tags: [architecture, capra-ui, frontend, data-source]
created: 2025-01-06
related: "[[ADR-UI-001 Adapter Pattern]], [[Filter Registry]]"
---

# Adapter Pattern (UI Layer)

> Camada de tradução entre componentes UI e fontes de dados heterogêneas.

## O que é
Uma interface única (`DataAdapter`) com implementações concretas (BIMachine, Mock, futuras REST APIs). Componentes consomem dados via adapter sem conhecer o backend.

## Por que existe
Sem ele, componentes ficam acoplados a uma fonte específica. Trocar de backend ou criar testes exige mudar componentes. Com ele, basta swap do adapter.

## Como funciona
- Interface `DataAdapter` define o contrato:
  - `fetchKpi(queryId): Promise<KpiData>`
  - `fetchList(queryId): Promise<ListData>`
  - `getFilters(): Filter[]`
  - `applyFilter(id, members): void`
  - `executeRaw(mdx, options): Promise<RawResult>` — controle explícito (filterIds, noFilters, objectConfig)
- Adapters concretos: `MockAdapter` (delay configurável), `BIMachineAdapter` (ReduxStore + MDX), futuro `RestAdapter`.
- `createAdapter()` faz detecção automática por ambiente.

## Relações
- `[[ADR-UI-001 Adapter Pattern]]` — decisão original
- `[[ADR-UI-009 Phase 3 Service Integration]]` — adição de `executeRaw`
- `[[Filter Registry]]` — usa adapter para sincronizar filtros entre schemas
- `[[Action Bus]]` — todas chamadas passam pelo adapter, despachadas pelo bus

## Exemplos concretos
Em testes, MockAdapter retorna dados estáticos com delay simulado. Em prod, BIMachineAdapter monta queries MDX e conversa via ReduxStore. Componentes não sabem a diferença.
