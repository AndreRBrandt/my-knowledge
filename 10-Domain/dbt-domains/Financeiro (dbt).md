---
type: domain
status: living
project: [capra-dbt]
tags: [dbt, financeiro, domain]
created: 2026-04-15
related: "[[dbt Architecture]], [[capra-dbt]], [[Vendas (dbt)]]"
---

# Financeiro (dbt)

> Domain financeiro — contas a pagar/receber, fluxo de caixa, DRE consolidado.

## Modelos

| Modelo | Tipo | Grão | Uso |
|--------|------|------|-----|
| `fct_contas_detalhe` | fact | Linha por conta (AP/AR) | Análise detalhada contas |
| `fct_contas_resumo` | fact | Agregado de contas | KPIs financeiros |
| `fct_fluxo_financeiro` | fact | Movimentação | Fluxo de caixa |
| `obt_financeiro` | OBT | Consolidado | Consumo API geral |
| `obt_financeiro_macro` | OBT | Visão macro | Relatórios executivos |

## Conceitos

### Contas a Pagar (AP)
Obrigações financeiras da empresa (fornecedores, serviços, impostos).

### Contas a Receber (AR)
Valores a receber de clientes.

### Fluxo Financeiro
Movimentação real de dinheiro (caixa, banco, PIX, cartão).

## Raw sources

- `stg_contapagar`, `stg_tipoctpagar` — AP
- `stg_contareceber`, `stg_tipoctreceber` — AR
- `stg_movfluxo` — fluxo de caixa
- `stg_classfinanc` — classificação financeira (plano de contas)
- `stg_caixa`, `stg_movcaixa`, `stg_turcaixa` — operações de caixa
- `stg_tiposangria` — tipos de sangria
- `stg_logoperfos` — log de operações (fiscal)

## Relações cross-domain

- **← Vendas:** pagamentos de vendas entram no fluxo (`int_pagamentos_venda`)
- **→ Core:** dim_filial pra segmentação
