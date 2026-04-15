---
type: domain
status: living
project: [capra-dbt]
tags: [dbt, avaliacoes, customer-feedback, domain]
created: 2026-04-15
related: "[[dbt Architecture]], [[capra-dbt]]"
---

# Avaliacoes (dbt)

> Domain de avaliações e feedback de clientes (satisfação, reviews, NPS).

## Modelos

| Modelo | Tipo | Grão | Uso |
|--------|------|------|-----|
| `fct_avaliacoes_detalhe` | fact | Por avaliação | Drill-down em feedback individual |
| `fct_avaliacoes_resumo` | fact | Agregado | KPIs (média de estrelas, NPS) |

## Fontes

### Avalyo
Plataforma de avaliações via Wifire (rede WiFi do local).
- `stg_avalyo_reviews` — reviews coletadas

### Wifire Ratings
Ratings capturados no portal captive do WiFi.
- `stg_wifire_ratings`

## Conceitos

### Avaliação
Uma nota (tipicamente 1-5) + comentário opcional, vinculada a uma filial (e às vezes a uma visita/venda específica via timestamp).

### NPS (Net Promoter Score)
Métrica derivada: % de promotores (9-10) − % de detratores (0-6). Calculado em `fct_avaliacoes_resumo`.

## Relações cross-domain

- **→ Core:** dim_filial pra segmentar por unidade
- **Possível cruzamento futuro:** correlacionar avaliação com venda específica (mesmo horário/filial)
