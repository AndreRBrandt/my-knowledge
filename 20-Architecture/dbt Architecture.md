---
type: architecture
status: living
project: [capra-dbt]
tags: [dbt, data-pipeline, medallion, supabase, postgres]
created: 2026-04-15
related: "[[Medallion Architecture]], [[Data Flow]], [[capra-dbt]]"
---

# dbt Architecture

> Arquitetura do pipeline de dados do Capra: **raw → staging → intermediate → marts (gold)** implementado em dbt rodando sobre Postgres (Supabase hoje, self-hosted como target).

## Visão geral

```
External APIs (Teknisa, PontoMais, iFood, ...)
       ↓ (ingestion scripts em projects_script/)
   Raw Tables
   (Postgres)
       ↓
   Staging (views, schema: staging)
   stg_* — limpeza e tipagem
       ↓
   Intermediate (views, schema: staging)
   int_* — joins e lógica reutilizável
       ↓
   Marts (tables, schema: gold)
   fct_*, dim_*, obt_* — dados finais de consumo
       ↓
   Backend API (bode-api Rust / bode-analytics-api TS)
       ↓
   Frontend (capra-analytics + capra-ui)
```

## Localização e estrutura

**Projeto dbt canônico:**
`capra-workspace/capra-dbt/capra_dbt/`

```
capra_dbt/
├── dbt_project.yml
├── models/
│   ├── staging/           # 32 modelos stg_* + pontomais/
│   ├── intermediate/      # 3 modelos int_* + cmv/ + rh/
│   └── marts/
│       ├── core/          # dim_filial (dimensão compartilhada)
│       ├── avaliacoes/    # fct_avaliacoes_*
│       ├── cmv/           # fct_cmv_*
│       ├── estoque/       # (vazio — não implementado ainda)
│       ├── financeiro/    # fct_*, obt_*
│       ├── rh/            # obt_rh_diario
│       └── vendas/        # fct_vendas_*, obt_*
├── macros/
├── snapshots/
├── seeds/
├── tests/
├── analyses/
└── packages.yml           # inclui dbt_expectations
```

## Materialização

| Camada | Schema | Tipo | Justificativa |
|--------|--------|------|---------------|
| Staging | `staging` | `view` | Leve, sempre fresh, sem duplicar storage |
| Intermediate | `staging` | `view` | Idem staging — lógica reutilizável sem persistência |
| Marts | `gold` | `table` | Pesado computacionalmente, consumido com alta frequência pela API |

## Execução

**Não se roda `dbt run` direto em produção.** O dbt é usado na etapa de modelagem e testes, mas a atualização em produção passa por função SQL incremental.

### Comando
```bash
pnpm ops dbt:refresh              # Default: últimos 3 dias
pnpm ops dbt:refresh --days 7     # Customizado
pnpm ops dbt:refresh --apply-function  # Re-aplica refresh_gold_incremental.sql
```

### O que faz por baixo
- Script Node.js: `capra-workspace/scripts/ops/commands/dbt-refresh.mjs`
- Conecta no Postgres via `pg` (node-postgres)
- Chama função SQL `refresh_gold_incremental(days)` no banco
- A função atualiza apenas marts do período especificado

### Por quê function SQL + dbt?
- **dbt** gera o SQL das transformações (staging, intermediate, marts) + documentação + testes
- **Função SQL** encapsula a lógica de refresh incremental (janela de dias) de forma performática
- **Combina:** dbt pro desenvolvimento/qualidade, função SQL pro runtime

## Convenções de nomenclatura

| Prefixo | Camada | Exemplo | Descrição |
|---------|--------|---------|-----------|
| `stg_` | staging | `stg_venda` | Clean view de uma tabela raw |
| `int_` | intermediate | `int_pagamentos_venda` | Lógica intermediária, joins |
| `dim_` | marts/core | `dim_filial` | Dimensão compartilhada |
| `fct_` | marts/domain | `fct_vendas_diario` | Fato (grão específico) |
| `obt_` | marts/domain | `obt_vendas_item` | One Big Table (consumo direto) |

## Dimensões

**Compartilhadas em `marts/core/`:**
- `dim_filial` — mestre de filiais (usada por todos os domínios)

_(Outras dimensões ficam inline nos marts de cada domínio ou são reaproveitadas via `ref()` do dbt.)_

## Domínios (marts)

Ver `[[10-Domain]]` MOC para detalhes de cada domínio. Resumo:

| Domínio | Arquivos | Status |
|---------|----------|--------|
| `core` | `dim_filial` | Ativo |
| `vendas` | `fct_vendas_diario`, `obt_cancelamentos`, `obt_vendas_item` | Ativo |
| `financeiro` | 5 modelos (contas, fluxo, obt) | Ativo |
| `cmv` | 3 modelos (diário, ficha técnica, produto diário) | Ativo |
| `avaliacoes` | 2 modelos (detalhe, resumo) | Ativo |
| `rh` | `obt_rh_diario` | Ativo (em expansão) |
| `estoque` | — | **Não implementado** |

## Roadmap

- **Imediato:** implementar domain `estoque`
- **Curto prazo:** expandir marts de RH conforme refatoração do pipeline PontoMais (ver `_legacy_reference/rh_pontomais_gold_layer/`)
- **Médio prazo:** migrar banco de Supabase → self-hosted Postgres (DNS swap, ver `[[ADR-005 DB Abstraction]]`)
- **Longo prazo:** reescrever ingestion pipeline em Rust junto com bode-api

## Decisões relacionadas

- `[[GADR-002 Silver Gold Separation]]` — Silver faz trabalho pesado, Gold faz embalagem
- `[[ADR-005 DB Abstraction]]` — Zero vendor lock-in, Supabase → self-hosted
- `[[ADR-009 Zero Vendor Lock-in]]` — Estratégia geral de neutralidade

## Referência obsoleta

⚠️ `bi_projects/docs/ARQUITETURA_DBT.md` na raiz era a documentação original deste pipeline mas foi escrita em um momento híbrido com BIMachine. **Esta nota no vault é a fonte canônica atual.**

## O que NÃO é dbt

- `projects_script/teknisa_crawler/` — extração da API Teknisa (ingestion)
- `projects_script/ifood-crawler/` — extração iFood (ingestion)
- `_legacy_reference/rh_pontomais_gold_layer/` — estrutura antiga PontoMais + BIMachine (descontinuado)
- `refresh_gold_incremental.sql` — função SQL manual que orquestra refresh (complementa dbt, não é gerada por ele)
