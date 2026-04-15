---
type: domain
status: living
project: [capra-dbt]
tags: [dbt, vendas, domain]
created: 2026-04-15
related: "[[dbt Architecture]], [[capra-dbt]], [[CMV (dbt)]]"
---

# Vendas (dbt)

> Domain de vendas — transações comerciais, itens vendidos, cancelamentos.

## Modelos

| Modelo | Tipo | Grão | Uso |
|--------|------|------|-----|
| `fct_vendas_diario` | fact | Uma linha por venda por dia | Análises agregadas por dia/filial |
| `obt_vendas_item` | OBT | Item de venda detalhado | Consumo direto pela API/frontend |
| `obt_cancelamentos` | OBT | Por cancelamento | Análises de cancelamento separadas |

## Conceitos

### Venda
Transação comercial completa: PDV, operador, cliente, data, itens, pagamento.

### Item de venda
Linha da venda — produto específico, quantidade, preço unitário, desconto aplicado.

### Cancelamento
Uma venda pode ser cancelada total ou parcialmente. Tracked separadamente em `obt_cancelamentos` por decisão arquitetural — ver `[[GADR ADR Tknisa Cancelamentos]]`.

## Raw sources

Staging usado pra compor esse mart:
- `stg_venda` — cabeçalho de vendas
- `stg_vendaitem` — itens de venda
- `stg_itvendacan` — itens cancelados
- `stg_itvendaimpos` — impostos de item
- `stg_cupomdescfos`, `stg_usocupomdescfos` — cupons de desconto
- `stg_operador`, `stg_vendedor` — quem fez a venda
- `stg_produto`, `stg_grupprod`, `stg_subgrupprod` — o que foi vendido

## Intermediate

- `int_pagamentos_venda` — consolida formas de pagamento por venda (usado aqui e em financeiro)

## Relações cross-domain

- **→ CMV:** cada item vendido consome ingredientes (ficha técnica) — ver `[[CMV (dbt)]]`
- **→ Financeiro:** pagamentos de vendas alimentam fluxo de caixa
- **→ Core:** dim_filial para segmentação
