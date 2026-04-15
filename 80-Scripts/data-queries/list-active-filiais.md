---
type: script
status: draft
project: [capra-dbt]
tags: [sql, filial, query]
created: 2026-04-15
category: data-query
parameters: []
related: "[[Core (dbt)]]"
---

# list-active-filiais

> Retorna todas as filiais ativas do grupo com código, nome e marca.

## When to use
- Precisar de lista atualizada de filiais ativas
- Validar se novas filiais foram adicionadas
- Mapear código ↔ nome (ex: "0002" é qual filial?)

**Read-only.** Não muta nada.

## Pre-requisites
- Acesso ao Supabase (credenciais em `.env.ops`)
- `psql` ou qualquer cliente SQL

## Query

```sql
SELECT
  codigo,
  nome,
  marca,
  status,
  data_abertura
FROM gold.dim_filial
WHERE status = 'ATIVO'
ORDER BY codigo;
```

## Parameters

Nenhum.

## Expected output

### Success
Tipicamente 9 filiais:

| codigo | nome | marca | status | data_abertura |
|--------|------|-------|--------|---------------|
| 0001 | BDN Boa Viagem | BODE | ATIVO | 2015-... |
| 0002 | BDN Guararapes | BODE | ATIVO | ... |
| ... | | | | |

Marcas esperadas: `BODE`, `BURGUER`, `ITALIANO`.

### Failure modes
- **0 rows** — algo muito errado. Nenhuma filial ativa? Check se nome da tabela mudou.
- **Permission denied** — conecta com user correto (não anon).
- **Relation does not exist** — `dim_filial` não foi materializada. Rode `pnpm ops dbt:refresh`.

## How to invoke

### Via psql
```bash
psql "$DB_URL" -f queries/list-active-filiais.sql
```

### Via Claude
Me peça: "lista filiais ativas" e eu executo se tiver credenciais.

## Source location

- Query inline nesta nota (fonte canônica)
- Tabela origem: `gold.dim_filial` (dbt model em `capra-dbt/models/marts/core/dim_filial.sql`)

## Notes

- Filiais inativas (ex: lojas fechadas) ainda aparecem em `dim_filial` com `status != 'ATIVO'` — mantém histórico
- Marca é enum informal — verificar se nova marca foi adicionada antes de assumir as 3 existentes

## Related

- `[[Core (dbt)]]` — domain onde `dim_filial` vive
