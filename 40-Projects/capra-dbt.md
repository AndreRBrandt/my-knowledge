---
type: hub
status: living
project: [capra-dbt]
tags: [dbt, data-pipeline]
created: 2026-04-15
related: "[[dbt Architecture]], [[Medallion Architecture]]"
---

# capra-dbt

> Projeto dbt do Capra. Transforma dados raw em marts gold consumidos pelo backend, seguindo padrão medallion.

## Stack
- **dbt** (core)
- **Postgres** via Supabase (target atual) → self-hosted (futuro)
- **Node.js** (orquestração via `dbt-refresh.mjs`)
- **dbt_expectations** (testes de qualidade de dados)

## Architecture
- `[[dbt Architecture]]` — pipeline, camadas, execução
- `[[Medallion Architecture]]` — padrão conceitual

## Domain (marts)

Marts implementados em `models/marts/`:

| Domain | Status | Notas |
|--------|--------|-------|
| `[[Core (dbt)]]` | Ativo | dim_filial, dim_produto, dim_operador |
| `[[Vendas (dbt)]]` | Ativo | obt_vendas_item, fct_vendas_diario, fct_vendas_hora, obt_cancelamentos |
| `[[Financeiro (dbt)]]` | Ativo | obt_financeiro (c/ operador), obt_fechamento_caixa, contas, fluxo |
| `[[CMV (dbt)]]` | Ativo | fct_cmv_diario, ficha_tecnica, produto_diario |
| `[[Avaliacoes (dbt)]]` | Ativo | detalhe + resumo |
| `[[RH (dbt)]]` | Ativo | obt_rh_diario (em expansão) |
| `[[Estoque (dbt)]]` | **Não implementado** | Próximo a ser feito |

## Active decisions

- `[[GADR-002 Silver Gold Separation]]`
- `[[ADR-005 DB Abstraction]]` — Supabase → self-hosted
- `[[ADR-009 Zero Vendor Lock-in]]`

## External

- **Repo:** dentro de `capra-workspace` monorepo
- **Local:** `capra-workspace/capra-dbt/capra_dbt/`
- **Execução:** `pnpm ops dbt:refresh [--days N]`
- **Script:** `capra-workspace/scripts/ops/commands/dbt-refresh.mjs`
- **Função runtime:** `refresh_gold_incremental(days)` no Postgres

## Data Sources & Ingestion

dbt não faz ingestão, só transformação. Fontes e cargas:
- `[[Teknisa Oracle Data Map]]` — mapeamento completo de tabelas/relacionamentos/PKs
- `[[Data Pipeline Status]]` — estado de cada tabela (fresh, stale, gaps)
- `[[teknisa-crawler]]` — API Teknisa (QueryRunner + Webtoken)
- PontoMais — 13 tabelas RH sincronizadas
- Avalyo / Wifire — reviews e NPS externos

## Active Work
- (nenhuma sessão registrada ainda)

## Legacy / Reference

Não faz mais parte do pipeline ativo, mas útil pra consulta:
- `_legacy_reference/rh_pontomais_gold_layer/` — estrutura antiga (BIMachine) do Gold RH, referência pra refatoração
