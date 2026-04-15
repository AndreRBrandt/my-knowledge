---
type: domain
status: draft
project: [capra-dbt]
tags: [dbt, estoque, domain, pending]
created: 2026-04-15
related: "[[dbt Architecture]], [[capra-dbt]], [[CMV (dbt)]]"
---

# Estoque (dbt)

> Domain de **controle de estoque** — saldos, movimentações, rupturas, inventário.

## Status

⚠️ **NÃO IMPLEMENTADO.** Pasta `marts/estoque/` existe mas está **vazia**.

Próximo domain a ser implementado no `capra-dbt`.

## Conceitos (esperados quando implementado)

### Saldo
Quantidade disponível de um produto em uma filial em um instante. Tipicamente calculado como: `saldo_inicial + entradas − saídas`.

### Entradas
Compras, transferências recebidas, ajustes positivos de inventário.

### Saídas
Vendas (via ficha técnica — ver `[[CMV (dbt)]]`), transferências enviadas, perdas, ajustes negativos.

### Ruptura
Situação onde saldo chega a zero (ou abaixo do ponto de reposição) e produto fica indisponível pra venda.

## Raw sources potenciais

Ainda não confirmado, mas provavelmente:
- `stg_movestoque` (a criar?)
- `stg_inventario` (a criar?)
- Já existe: `stg_custocmpprod` (compras)

## Relações cross-domain previstas

- **← CMV:** consumo via ficha técnica de vendas abate saldo
- **→ Core:** dim_filial
- **→ Financeiro:** compras alimentam AP
