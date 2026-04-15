---
type: domain
status: living
project: [capra-dbt]
tags: [dbt, dimensions, domain-core]
created: 2026-04-15
related: "[[dbt Architecture]], [[capra-dbt]]"
---

# Core (dbt)

> Dimensões **compartilhadas** entre domínios. Não é um domain de negócio por si, mas infraestrutura dimensional que todos os outros domínios consomem.

## Modelos

| Modelo | Grão | Descrição |
|--------|------|-----------|
| `dim_filial` | Uma linha por filial | Mestre de filiais do grupo |

## Por que "core"

Esse é o único mart onde **múltiplos domínios compartilham** a mesma dimensão. Manter filial aqui (e não duplicar em cada mart) garante:
- Single source of truth de metadados de filial
- Joins consistentes cross-domain
- Mudança de filial propaga automaticamente

## Relacionado

- Raw source: `stg_filial`, `stg_loja`
- Consumido por: todos os outros marts (vendas, financeiro, cmv, rh, avaliacoes)
