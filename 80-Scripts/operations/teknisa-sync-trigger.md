---
type: script
status: stable
project: [bode-analytics-api]
tags: [teknisa, sync, api, trigger]
created: 2026-04-16
category: operation
parameters: [environment]
related: "[[Data Pipeline Status]]"
---

# teknisa-sync-trigger

> Dispara todas as rotas de sync da API (50+ rotas). Orquestra carga Teknisa → Supabase via webtokens.

## When to use
- Sync automático (cron) parou
- Após renovar webtokens
- Carga em massa após manutenção

**MUTAÇÃO.** Trigera upserts em todas as tabelas raw do Supabase.

## Pre-requisites
- API rodando (prod ou homolog)
- Credenciais Supabase (email/password pra auth)
- Webtokens válidos configurados nos secrets da API

## Command

```bash
cd capra-workspace

# Produção
node scripts/trigger_all_syncs.mjs

# Homolog
node scripts/trigger_all_syncs.mjs --homolog
```

## Parameters

| Param | Type | Default | Description |
|---|---|---|---|
| `--homolog` | flag | false | Aponta pra API homolog em vez de prod |

## Expected output

### Success
```
Autenticando... OK
Triggering 52 sync routes...
  raw.venda: 2450 rows synced (3.2s)
  raw.vendaitem: 8500 rows synced (8.1s)
  ...
  TOTAL: 45 OK, 3 SKIPPED, 4 ERROR
```

### Failure modes
- **401 Unauthorized** — credenciais Supabase inválidas ou expiradas
- **Webtoken expired** — route-specific. Checar `sync_log` pra identificar quais
- **API down** — verificar container com `[[check-vps-health]]`

## Source location
- Script: `capra-workspace/scripts/trigger_all_syncs.mjs`
- Rotas de sync: `bode-analytics-api/src/routes/sync-raw-*.ts` (28+ rotas)
- Sync lib: `bode-analytics-api/src/lib/create-raw-sync.ts`

## Notes
- Sync automático via cron: `10 * * * *` (transacionais a cada hora), `0 5 * * *` (dimensões 1x/dia)
- Workflow GitHub `sync-teknisa.yml` no repo bodedono/bode-analytics faz trigger via schedule
- **Status atual (2026-04-16):** sync parado pra movcaixa, itvendacan, usocupomdescfos (webtokens expirados)
