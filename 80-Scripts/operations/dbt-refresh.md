---
type: script
status: stable
project: [capra-dbt]
tags: [dbt, refresh, gold, incremental]
created: 2026-04-15
category: operation
parameters: ["--days N", "--apply-function"]
related: "[[dbt Architecture]], [[capra-dbt]]"
---

# dbt-refresh

> Atualiza marts (gold) no Supabase chamando a função SQL `refresh_gold_incremental`. Default: últimos 3 dias.

## When to use
- Após ingestão de novos dados raw (Teknisa, PontoMais, etc)
- Pra garantir marts refletem transações recentes
- Após deploy de novo modelo dbt (reprocessar histórico)
- Manualmente em troubleshooting de divergência gold vs raw

⚠️ **Script mutativo** — altera dados em tabelas gold.

## Pre-requisites
- `.env.ops` configurado com `DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`
- `pg` (node-postgres) instalado em `bode-analytics-api/node_modules`
- Conexão com Supabase (VPN se necessário)

## Command

```bash
# Default: últimos 3 dias
pnpm ops dbt:refresh

# Customizado: últimos N dias
pnpm ops dbt:refresh --days 7

# Re-aplicar a função SQL (útil se mudou o SQL do refresh)
pnpm ops dbt:refresh --apply-function
```

## Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `--days N` | int | 3 | Janela de refresh incremental (dias atrás) |
| `--apply-function` | flag | false | Re-executa `refresh_gold_incremental.sql` (DDL da function) |

## Expected output

### Success
```
✓ Connected to database: aws.{project}.supabase.co
✓ Refreshing gold layer for last 3 day(s)...
✓ refresh_gold_incremental(3) executed successfully
✓ Duration: 12.3s
✓ Done
```

### Failure modes
- **`Credenciais faltando`** — `.env.ops` não tem DB_*. Configure.
- **`pg module não encontrado`** — rode `cd bode-analytics-api && pnpm install`
- **`connection refused`** — Supabase inacessível. Check VPN/IP whitelist.
- **`function refresh_gold_incremental does not exist`** — função não foi criada. Rode com `--apply-function`.

## How to invoke

```bash
cd capra-workspace
pnpm ops dbt:refresh --days 3
```

## Source location

- Script: `capra-workspace/scripts/ops/commands/dbt-refresh.mjs`
- SQL function: definida em migrations do Supabase (`refresh_gold_incremental.sql`)
- Config: `capra-workspace/.env.ops`

## Notes

- A função SQL no banco é **independente** do dbt. dbt gera os modelos; a função orquestra o refresh.
- Refresh é **incremental por data** — usa coluna `data_movimento` (ou equivalente) de cada mart.
- Pra full refresh (reprocessar TUDO): `--days 9999` ou rodar `dbt run --full-refresh` direto (raro, pesado).

## Related

- `[[dbt Architecture]]` — como o pipeline é estruturado
- `[[capra-dbt]]` — projeto dbt
