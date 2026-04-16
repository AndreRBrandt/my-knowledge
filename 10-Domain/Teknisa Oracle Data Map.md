---
type: concept
status: living
project: [capra-dbt, bode-api]
tags: [domain, teknisa, oracle, data-map, tables]
created: 2026-04-16
---

# Teknisa Oracle Data Map

> Mapeamento completo do banco Oracle Teknisa (USR_ORG_5292) — tabelas, volumes, PKs, relacionamentos e status de sync.

## Conexao

- **Host:** Oracle via Teknisa QueryRunner (`bi.teknisa.com`)
- **NRORG:** 5292 (Grupo do No)
- **Auth:** Zeedhi OAuth (email + password + product_id + oauth_hash)
- **Script:** `projects_script/teknisa_crawler/` (Python)
- **Connection ID:** `6740aabb05a5c63f7b479865`
- **DB URL:** `jdbc:oracle:thin:@USR_ORG_5292`

## Tabelas Transacionais (fatos)

| Tabela | PK | Rows (Apr/2026) | DATA_REF | Freq Sync | Status |
|---|---|---:|---|---|---|
| VENDA | cdfilial, cdcaixa, nrseqvenda | 600.758 | DTENTRVENDA | hourly | OK |
| ITEMVENDA (vendaitem) | cdfilial, cdcaixa, nrseqvenda, nrsequitvend | 2.621.986 | herda VENDA.DTENTRVENDA | hourly | OK |
| ITVENDAIMPOS | cdfilial, cdcaixa, nrseqvenda, nrsequitvend, nrseqitimpos | 2.621.986 | herda VENDA.DTENTRVENDA | hourly | Sync ok, **NAO usado no dbt** |
| ITVENDACAN | cdfilial, cdcaixa, nrseqvenda, nrseqitvendc | 35.065 | DTHRPRODCAN | hourly | **20% desatualizado** |
| MOVCAIXA | cdfilial, cdcaixa, nrseqvenda, nrsequmovi | 1.072.067 | DTMOVIMCAIXA | hourly | Sync stale, carga manual feita (7d) |
| TURCAIXA | cdfilial, cdcaixa, dtabercaix | 26.247 | DTMOVTURCAIX | hourly | ~OK (-2.5%) |
| LOGOPERFOS | sem PK natural (duplicatas reais) | 100.489 | DTHROPERFOS | hourly | Sync ok, **NAO usado no dbt** |
| USOCUPOMDESCFOS | cdfilial, cdcaixa, nrseqvenda, cdcupomdescfos | 13.063 | herda VENDA.DTENTRVENDA | hourly | **69% desatualizado** |
| CONTAPAGAR | compound PK | 61.074 | dtvenctopaga | daily | -5% gap |
| CONTARECEBER | compound PK | 326.099 | dtvenctoreceb | daily | **-7% gap** |
| MOVFLUXO | compound PK | ~14K | dtmovimento | daily | OK |
| CUSTOCMPPROD | cdfilial, cdalmoxarife, cdlocalestoq, nrloteestq, nrsublote, cdproduto, dtcustcmppro | 50.450 | snapshot | daily | ~OK (-2%) |
| ROTFABRICA | cdproduto, cdsubproduto, cdgrupocomp, nrseqprod | 788 | snapshot | daily | OK |

## Dimensoes

| Tabela | PK | Rows | Status |
|---|---|---:|---|
| FILIAL | cdfilial | 9 | OK |
| LOJA | cdloja | ~20 | OK |
| CAIXA | cdfilial, cdcaixa | ~50 | OK |
| OPERADOR | cdoperador | 432 | -15 gap |
| VENDEDOR | cdvendedor | 306 | -19 gap |
| CONSUMIDOR | cdconsumidor | 4.756 | ~OK |
| PRODUTO | cdproduto | 7.784 | ~OK (dedup por LENGTH) |
| TIPOSANGRIA | cdfilial, cdtpsangria | 74 | OK |
| TIPORECE | cdtiporece | 57 | OK |
| TIPOCONS | cdtipocons | 13 | OK |
| GRUPPROD | cdgrupprod | 10 | OK |
| SUBGRUPPROD | cdgrupprod, cdsubgrprod | 41 | OK |
| GRUPROMOC | cdgrupromoc | ~50 | OK |
| CAMPANHAPROMO | cdcamppromo | ~20 | OK |
| CUPOMDESCFOS | cdcupomdescfos | ~100 | OK |
| EMPRESA | cdempresa | ~5 | **NAO no dbt** |

## Relacionamentos Principais

```
VENDA (header)
 |-- ITEMVENDA (itens da venda, JOIN: cdfilial+cdcaixa+nrseqvenda)
 |    |-- ITVENDAIMPOS (impostos por item, JOIN: +nrsequitvend)
 |    |-- PRODUTO (dimensao, JOIN: cdproduto)
 |    |    |-- GRUPPROD (JOIN: cdgrupprod — esta em ITEMVENDA, NAO em PRODUTO)
 |    |    |-- SUBGRUPPROD (JOIN: cdgrupprod+cdsubgrprod)
 |    |-- VENDEDOR (JOIN: cdvendedor)
 |    |-- USOCUPOMDESCFOS (cupons usados, JOIN: cdfilial+cdcaixa+nrseqvenda)
 |         |-- CUPOMDESCFOS (catalogo, JOIN: cdcupomdescfos)
 |         |-- GRUPROMOC (grupo promo, JOIN: cdgrupromoc)
 |         |-- CAMPANHAPROMO (campanha, JOIN: cdcamppromo)
 |-- ITVENDACAN (itens cancelados, JOIN: cdfilial+cdcaixa+nrseqvenda)
 |-- OPERADOR (JOIN: cdoperador)
 |-- CONSUMIDOR (JOIN: cdconsumidor)
 |-- FILIAL (JOIN: cdfilial)
 |    |-- LOJA (JOIN: cdloja — dentro de filial)
 |    |-- EMPRESA (JOIN: cdempresa)
 |-- CAIXA (JOIN: cdfilial+cdcaixa)

MOVCAIXA (movimentos de caixa)
 |-- TIPOSANGRIA (JOIN: cdfilial+cdtpsangria — apenas tipo G)
 |-- TIPORECE (JOIN: cdtiporece — tipo de recebimento)
 |-- OPERADOR (JOIN: cdoperador)
 |-- CAIXA (JOIN: cdfilial+cdcaixa)
 |-- FILIAL (JOIN: cdfilial)

TURCAIXA (turnos de caixa)
 |-- OPERADOR abertura (JOIN: cdoperaber)
 |-- OPERADOR fechamento (JOIN: cdoperfech)
 |-- CAIXA (JOIN: cdfilial+cdcaixa)
 |-- FILIAL (JOIN: cdfilial)

CUSTOCMPPROD (custo de ingredientes)
 |-- PRODUTO (JOIN: cdproduto — ingrediente)
 |-- FILIAL (JOIN: cdfilial)

ROTFABRICA (receitas / ficha tecnica)
 |-- PRODUTO raiz (JOIN: cdproduto — produto final)
 |-- PRODUTO ingrediente (JOIN: cdsubproduto → cdproduto)
 |-- Recursiva: ingrediente pode ser outro produto com sub-receita (depth < 5)
```

## Tipos de Movimento MOVCAIXA (IDTIPOMOVIVE)

| Tipo | Descricao | Volume | Uso no BI |
|---|---|---|---|
| A | Abertura de caixa (saldo inicial) | 24K | Excluido de obt_financeiro (nao e movimento real) |
| E | Entrada (pagamento em venda) | 777K | Excluido quando tem nr_seq_venda (coberto por int_pagamentos_venda) |
| G | Sangria (retirada de caixa) | 129K | **INCLUIDO** — classificado por TIPOSANGRIA (F=Fechamento, D=Despesa) |
| S | Troco (saida de dinheiro) | 17K | INCLUIDO como SAIDA_DINHEIRO |
| T | Debito (pagamento em venda) | 5.8K | Excluido quando tem nr_seq_venda |
| C | Cancelamento manual | 213 | INCLUIDO |

## Sub-classificacao de Sangrias (TIPOSANGRIA.IDSANGRIA)

| IDSANGRIA | Significado | Volume | Valor |
|---|---|---|---|
| F | Fechamento de turno | 109K | Sangria de fim de turno (valor padrao) |
| D | Despesa | 10K | Pagamentos diversos (vale, compras, uber, coleta, etc.) |
| NULL | Sem match | 13K | R$7M sem classificacao (gap no JOIN) |

## Tabelas NAO no Pipeline (Oracle Table Priorities)

### Onda 1 (254K rows, ~33MB)
| Tabela | Rows | PK | Desbloqueia |
|---|---:|---|---|
| FORNECEDOR | 1.750 | cdfornecedor | Analise fornecedores em contas a pagar |
| MESA | 2.520 | cdfilial, cdmesa | Analise por mesa (tempo, ocupacao) |
| BOLETPAGAR | 89K | compound | Boletos a pagar detalhados |
| BAIXARECEBIDA | 162K | compound | Baixas de recebimento |

### Onda 2 (597K rows)
| Tabela | Rows | Desbloqueia |
|---|---:|---|
| BOLETRECEBER | ~200K | Boletos a receber |
| PRODFILI | ~50K | Produto por filial (estoque) |
| PRODFORNCOMP | ~30K | Fornecedor por produto |

## Scripts de Acesso

### QueryRunner (SQL direto no Oracle)
```bash
cd projects_script/teknisa_crawler
python run_query.py queries/<domain>/<query_name>/query.sql
```
Config: `.env` com TEKNISA_BI_* credentials.

### Carga para Supabase (raw schema)
```bash
cd capra-workspace
# Via QueryRunner (dia a dia)
SUPABASE_URL=... SUPABASE_SERVICE_ROLE_KEY=... node scripts/load_raw_queryrunner.mjs <tabela> --from YYYY-MM-DD --to YYYY-MM-DD

# Via API Webtoken (bulk)
node scripts/fetch_api_to_csv.mjs <tabela>

# Trigger sync automatico (todas as rotas)
node scripts/trigger_all_syncs.mjs
```

### Script ad-hoc (Python direto)
```bash
cd projects_script/teknisa_crawler
python load_sangrias_to_supabase.py [dias]  # Carrega movcaixa + tiposangria
```

## Relacionado
- `[[capra-dbt]]` — pipeline de transformacao (raw → staging → gold → api)
- `[[Data Pipeline Status]]` — estado atual de cada tabela
- `[[Strangler Fig Pattern]]` — migracao TS → Rust
