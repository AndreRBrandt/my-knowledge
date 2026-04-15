---
type: concept
status: stable
project: [capra-ui]
tags: [architecture, capra-ui, organization, component-design]
created: 2025-01-08
related: "[[ADR-UI-002 Component Categories]], [[Domain Containers]], [[Core vs App Dependency Boundaries]]"
---

# Component Categories (UI vs Containers vs Analytics)

> Quatro categorias com hierarquia de dependência clara: cada componente pertence a UMA delas, e a direção de imports é unidirecional.

## O que é
Sistema de organização de componentes em pastas + regras de quem pode importar de quem.

```
src/core/components/
├── ui/          # Blocos básicos
├── containers/  # Wrappers estruturais
├── analytics/   # Exibem dados
└── layout/      # Estrutura de página
```

## Por que existe
Sem categorias, qualquer componente importa qualquer outro → grafo cíclico, impossível raciocinar sobre dependências, alterações em "ui" quebram "analytics" e vice-versa. Categorias forçam DAG.

## Como funciona

### Definições
| Categoria | Propósito | Exemplos |
|-----------|-----------|----------|
| `ui/` | Blocos básicos genéricos | BaseButton, Modal, ConfigPanel, Popover |
| `containers/` | Wrappers estruturais (sem domínio) | AnalyticContainer |
| `analytics/` | Exibem dados analíticos | KpiCard, DataTable, Charts |
| `layout/` | Estrutura geral da página | AppShell |

### Regras de import (DAG)
- 1 componente = 1 categoria
- `ui/` não importa de `analytics/` nem `containers/`
- `analytics/` pode usar `ui/` e `containers/`
- `layout/` pode usar qualquer

### Camada acima: Domain Containers
`KpiContainer`, `DataTableContainer` ficam acima de `analytics/` mas no app — encapsulam orquestração de domínio. Ver `[[Domain Containers]]`.

## Relações
- `[[ADR-UI-002 Component Categories]]` — decisão original
- `[[Core vs App Dependency Boundaries]]` — extensão da mesma filosofia entre framework e app
- `[[Domain Containers]]` — camada acima destas categorias
- Documentação espelha estrutura: `docs/components/{ui,containers,analytics,layout}/`

## Edge cases
Quando classificação é ambígua: priorizar a categoria mais "abaixo" no grafo (menos dependências). Se um componente combina UI + dados, refatorar em dois.
