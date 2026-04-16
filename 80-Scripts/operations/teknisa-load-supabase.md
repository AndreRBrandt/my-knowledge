---
type: script
status: stable
project: [capra-dbt]
tags: [teknisa, oracle, supabase, etl, carga]
created: 2026-04-16
category: operation
parameters: [tabela, from, to, dry-run]
related: "[[Teknisa Oracle Data Map]], [[Data Pipeline Status]]"
---

# teknisa-load-supabase

> Carrega dados do Oracle Teknisa → Supabase raw schema. Dia a dia pra transacionais, snapshot pra dimensões.

## When to use
- Dados stale no Supabase (gap entre Oracle e raw)
- Carga inicial de tabela nova
- Recarga após correção de dados na fonte

**MUTAÇÃO.** Insere/atualiza dados no Supabase raw schema.

## Pre-requisites
- Node.js 22+
- `.env.ops` com `SUPABASE_URL` + `SUPABASE_SERVICE_ROLE_KEY`
- Config QueryRunner em `tknisa_silver_layer/scripts/config.json` (ou ajustar QR_CONFIG_PATH)
- Python QueryRunner funcionando (script depende dele internamente)

## Command

### Via Node.js (tabelas pré-configuradas)
```bash
cd capra-workspace

# Dimensão (snapshot completo)
SUPABASE_URL=... SUPABASE_SERVICE_ROLE_KEY=... \
  node scripts/load_raw_queryrunner.mjs {tabela}

# Transacional (por período)
SUPABASE_URL=... SUPABASE_SERVICE_ROLE_KEY=... \
  node scripts/load_raw_queryrunner.mjs {tabela} --from {YYYY-MM-DD} --to {YYYY-MM-DD}

# Dry run (valida sem carregar)
node scripts/load_raw_queryrunner.mjs {tabela} --dry-run
```

### Via Python (ad-hoc, quando Node não tem a tabela configurada)
```bash
cd projects_script/teknisa_crawler
SUPABASE_URL=... SUPABASE_SERVICE_ROLE_KEY=... \
  python load_sangrias_to_supabase.py {dias}
```

## Parameters

| Param | Type | Default | Description |
|---|---|---|---|
| `tabela` | string | obrigatório | Nome da tabela (venda, vendaitem, movcaixa, tiposangria, etc) |
| `--from` | date | 30 dias atrás | Data início (YYYY-MM-DD) |
| `--to` | date | hoje | Data fim (YYYY-MM-DD) |
| `--dry-run` | flag | false | Valida contagens sem carregar |

### Tabelas configuradas no Node.js
venda, vendaitem, itvendaimpos, movcaixa, turcaixa, itvendacan, logoperfos, usocupomdescfos, custocmpprod, rotfabrica, grupromoc, campanhapromo, cupomdescfos, tiposangria, grupprod, subgrupprod, produto, filial, loja, caixa, operador, vendedor, consumidor, tipocons, tiporece

## Expected output

### Success
```
=== venda (2026-04-10 → 2026-04-16) ===
  Oracle validation: CNT=2450, SUM_TOT=1234567.89
  2026-04-10: 350 rows (upserted)
  2026-04-11: 380 rows (upserted)
  ...
  TOTAL: 2450 rows loaded
  Supabase validation: CNT=2450, SUM_TOT=1234567.89 ✓
```

### Failure modes
- **ENOENT config.json** — ajustar QR_CONFIG_PATH no script ou criar config
- **409 Conflict** — PK errada ou constraint mismatch. Usar delete+insert strategy
- **500 Supabase** — tabela não existe no raw schema. Criar via migration.
- **0 rows Oracle** — NRORG errado, data sem dados, ou tabela renomeada

## Source location
- Node.js: `capra-workspace/scripts/load_raw_queryrunner.mjs`
- Python (sangrias): `projects_script/teknisa_crawler/load_sangrias_to_supabase.py`
- PK definitions: linhas 28-350 do `load_raw_queryrunner.mjs`

## Notes
- Estratégia pra transacionais: dia a dia (evita timeout do QueryRunner em queries grandes)
- Estratégia pra dimensões: snapshot completo (delete all + insert, ou upsert por PK)
- Sempre comparar contagem Oracle vs Supabase pós-carga
- BATCH_SIZE = 500 rows por request ao Supabase REST API
- Consultar `[[Teknisa Oracle Data Map]]` pra PKs corretas antes de criar nova tabela
