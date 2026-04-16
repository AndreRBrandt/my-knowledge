---
type: script
status: stable
project: [bode-analytics-api]
tags: [teknisa, csv, export, webtoken]
created: 2026-04-16
category: data-query
parameters: [tabela, all]
related: "[[Teknisa Oracle Data Map]]"
---

# teknisa-fetch-csv

> Baixa dados da API Teknisa (via webtoken) e salva em CSV local. Sem banco — só arquivo.

## When to use
- Explorar dados antes de criar sync pipeline
- Backup pontual de tabela
- Validar formato/colunas da API
- Comparação offline (Excel/Power BI)

**Read-only.** Não muta banco.

## Pre-requisites
- Node.js 22+
- `.env.api` ou env vars com webtokens (TEKNISA_RAW_{TABELA}_WEBTOKEN)

## Command

```bash
cd capra-workspace

# Tabela específica
node scripts/fetch_api_to_csv.mjs {tabela}

# Todas as tabelas configuradas
node scripts/fetch_api_to_csv.mjs --all
```

## Parameters

| Param | Type | Default | Description |
|---|---|---|---|
| `tabela` | string | obrigatório | Nome da tabela (venda, vendaitem, movcaixa, etc) |
| `--all` | flag | false | Baixa todas as 28+ tabelas |

## Expected output

### Success
```
Fetching venda...
  → scripts/tmp/venda_api_20260416_143000.csv (2450 rows, 1.2 MB)
```
Arquivos em `scripts/tmp/`.

### Failure modes
- **401/403** — webtoken expirado. Renovar na Teknisa ZMart.
- **Timeout** — tabela grande. API tem limite interno.

## Source location
- Script: `capra-workspace/scripts/fetch_api_to_csv.mjs`
- Webtokens: `bode-analytics-api/.env` (variáveis TEKNISA_RAW_*_WEBTOKEN)
