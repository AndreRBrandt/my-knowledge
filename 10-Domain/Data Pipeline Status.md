---
type: concept
status: living
project: [capra-dbt, bode-api]
tags: [domain, pipeline, sync, data-quality]
created: 2026-04-16
---

# Data Pipeline Status

> Estado atual do pipeline de dados: Oracle → raw (Supabase) → staging → gold → API views. Atualizado em 2026-04-16.

## Pipeline Overview

```
Teknisa Oracle (USR_ORG_5292)
    ↓  Sync (Webtoken API ou QueryRunner)
Supabase raw schema (~31 tabelas)
    ↓  dbt staging (views: cast + snake_case)
Supabase staging schema (~68 views)
    ↓  dbt intermediate (tabelas: joins complexos, CMV recursivo)
Supabase staging schema (~6 tabelas)
    ↓  dbt gold/marts (tabelas: desnormalizadas, indexadas)
Supabase gold schema (~20 tabelas)
    ↓  dbt api (views: contratos para Workers API)
Supabase public schema (~23 views)
    ↓  Workers API (/api/query)
Frontend (capra-analytics)
```

## Status por Tabela (2026-04-16)

### Atualizado (sync funcionando)
- filial, loja, caixa, tiporece, tipocons, tiposangria, grupprod, subgrupprod
- produto, rotfabrica, custocmpprod
- contapagar, contareceber, movfluxo
- avalyo_reviews, wifire_ratings
- PontoMais (13 tabelas RH)

### Desatualizado (sync parou ou gap significativo)
| Tabela | Oracle | Supabase | Gap | Impacto |
|---|---:|---:|---|---|
| usocupomdescfos | 13.063 | 3.989 | **69%** | Dados de cupom/desconto incompletos |
| itvendacan | 35.065 | 27.906 | **20%** | Cancelamentos faltando |
| contareceber | 326.099 | 303.496 | **7%** | Contas a receber incompletas |
| operador | 432 | 417 | 3.5% | 15 operadores novos nao sincronizados |
| vendedor | 306 | 287 | 6% | 19 vendedores novos |

### Sync quebrado (webtoken expirado ou erro)
- movcaixa: sync automatico parado, carga manual de 7 dias feita em 2026-04-16
- Causa provavel: webtokens TEKNISA_RAW_MOVCAIXA_WEBTOKEN expirado

### Sincronizado mas NAO usado pelo dbt
| Tabela | Rows | Potencial |
|---|---:|---|
| itvendaimpos | ~2.6M | Impostos por item — analise fiscal/DRE |
| logoperfos | 352K | Operacoes forcadas PDV — **anti-fraude** |
| empresa | ~5 | Dimensao empresa/marca |

## API Views Disponiveis (public schema)

### Vendas
- `vendas` — item-level (2.5M rows)
- `api_vendas_resumo` — diario agregado (13K rows)
- `api_vendas_hora` — por hora/filial/dia (45K rows) **[NOVO 2026-04-16]**
- `api_delivery` — delivery por plataforma **[NOVO 2026-04-16]**

### Financeiro
- `financeiro` — movimentos de caixa (150K rows, agora com operador)
- `api_sangrias` — sangrias filtradas **[NOVO 2026-04-16]**
- `api_fechamento_caixa` — turnos de caixa (25K rows) **[NOVO 2026-04-16]**
- `api_financeiro_macro` — agregado alto nivel
- `api_fluxo_financeiro` — fluxo de caixa
- `api_contas_detalhe` + `api_contas_resumo` — AP/AR

### Dimensoes
- `api_produtos` — catalogo (7.5K) **[NOVO 2026-04-16]**
- `api_grupos_produto` — grupos e subgrupos **[NOVO 2026-04-16]**
- `api_operadores` — catalogo (417) **[NOVO 2026-04-16]**

### Outros
- `cancelamentos_itens` — cancelamentos (28K)
- `api_cmv_diario` — CMV por filial/dia (3.7K)
- `api_cmv_produto` + `api_cmv_ficha_tecnica` — CMV detalhado
- `api_avaliacoes_*` — reviews/NPS
- `api_rh_diario` — banco de horas
- `api_ads_daily` — campanhas ads
- `api_feriados` — calendario

## Acoes Pendentes

### Urgente
- [ ] Renovar webtokens expirados na Teknisa (movcaixa, itvendacan, usocupomdescfos)
- [ ] Reconstruir sync no bode-api (Rust) com retry, logs, alertas
- [ ] Rodar ANALYZE nas tabelas gold/public (pg_stat desatualizado)

### Curto prazo
- [ ] Enriquecer obt_vendas_item com CMV por item (remover stubs vr_cmv_item=0)
- [ ] Integrar logoperfos no pipeline (anti-fraude)
- [ ] Carregar Onda 1: FORNECEDOR, MESA, BOLETPAGAR, BAIXARECEBIDA

### Medio prazo
- [ ] Integrar itvendaimpos (impostos) no pipeline
- [ ] Carregar Onda 2: BOLETRECEBER, PRODFILI, PRODFORNCOMP
- [ ] Ativar sync de estoque (estoque_posicao, estoque_movimentos)

## Relacionado
- `[[Teknisa Oracle Data Map]]` — mapeamento de tabelas e relacionamentos
- `[[capra-dbt]]` — projeto dbt
- `[[bode-api]]` — backend Rust (novo sync pipeline)
