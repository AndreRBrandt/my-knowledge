---
type: domain
status: living
project: [capra-dbt]
tags: [dbt, cmv, custos, domain]
created: 2026-04-15
related: "[[dbt Architecture]], [[capra-dbt]], [[Vendas (dbt)]]"
---

# CMV (dbt)

> Domain de **Custo da Mercadoria Vendida**. Calcula quanto custou produzir o que foi vendido, baseado em ficha técnica + custo histórico dos insumos.

## Modelos

| Modelo | Tipo | Grão | Uso |
|--------|------|------|-----|
| `fct_cmv_diario` | fact | Por dia + filial | CMV consolidado |
| `fct_cmv_ficha_tecnica` | fact | Por ficha técnica | Análise por produto |
| `fct_cmv_produto_diario` | fact | Produto + dia | Análise granular |

## Intermediate

`intermediate/cmv/` — contém lógica complexa de:
- Resolução de ficha técnica (hierárquica)
- Cálculo de custo médio ponderado
- Timing entre compra e venda

## Conceitos

### Ficha técnica (receita)
Lista de insumos necessários pra produzir 1 unidade de um produto. Ex: hambúrguer = 1 pão + 1 patty + 2 alfaces + 1 queijo.

### CMV
CMV = Σ (quantidade consumida × custo unitário). O custo unitário é o custo médio ponderado de compra.

### Roteiro de fabricação
Processo de transformação (ficha técnica aplicada). Documento em `stg_rotfabrica`.

## Raw sources

- `stg_rotfabrica` — roteiros/fichas técnicas
- `stg_custocmpprod` — custo de compra de produtos
- `stg_produto`, `stg_grupprod`, `stg_subgrupprod` — produtos
- `stg_vendaitem` (via `[[Vendas (dbt)]]`) — o que foi vendido

## Relações cross-domain

- **← Vendas:** consumo de ingredientes parte do que foi vendido
- **→ Core:** dim_filial pra segmentação
