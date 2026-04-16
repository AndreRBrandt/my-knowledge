---
type: script
status: stable
project: [capra-dbt, bode-api]
tags: [teknisa, oracle, query-runner, sql]
created: 2026-04-16
category: data-query
parameters: [sql_file, sql_inline]
related: "[[Teknisa Oracle Data Map]]"
---

# teknisa-query-runner

> Executa SQL direto no Oracle Teknisa (USR_ORG_5292) via QueryRunner API. Retorna JSON/CSV.

## When to use
- Validar dados na fonte (contagens, amostras)
- Explorar tabelas novas antes de criar sync
- Comparar Oracle vs Supabase (auditoria)
- Diagnosticar gaps de dados

**Read-only.** Não muta nada no Oracle.

## Pre-requisites
- Python 3.10+
- `.env` em `projects_script/teknisa_crawler/` com credenciais Teknisa BI
- Sessão válida (auto-renew a cada 4h)

## Command

### Via arquivo SQL
```bash
cd projects_script/teknisa_crawler
python run_query.py queries/{domain}/{query_name}/query.sql
```
Output: terminal (primeiras 5 rows) + `queries/{domain}/{query_name}/amostra.csv`

### Via Python inline
```python
cd projects_script/teknisa_crawler
python -c "
from src.query_runner import run_query
rows = run_query(\"SELECT * FROM {TABELA} WHERE NRORG = 5292 AND ROWNUM <= {N}\")
for r in rows: print(r)
"
```

### Via módulo (importável)
```python
from src.auth import get_session
from src.query_runner import run_query

session = get_session()  # cached 4h
rows = run_query(sql, session)
```

## Parameters

| Param | Type | Default | Description |
|---|---|---|---|
| `sql_file` | path | - | Caminho pra arquivo .sql |
| `sql_inline` | string | - | SQL direto (alternativa ao arquivo) |
| `NRORG` | int | 5292 | Sempre filtrar por org (obrigatório) |

## Expected output

### Success
```
[AUTH] Usando sessao bi em cache (idade: 0.3h)
[QUERY] Executando query (N chars)...
[QUERY] Retornou M linhas
{'CDFILIAL': '0001', 'NMFILIAL': 'Bode do Nô - Boa Viagem', ...}
```

### Failure modes
- **0 linhas com NRORG=1** — NRORG errado. Usar 5292.
- **Session expired** — auto-renew falha se senha mudou. Verificar `.env`.
- **Timeout** — query muito pesada. Adicionar `ROWNUM <= N` ou filtro de data.
- **Aspas simples** — dentro de Python string, escapar: `''` (Oracle) não `\'`

## Source location
- Script: `bi_projects/projects_script/teknisa_crawler/`
- Auth: `src/auth.py` (Zeedhi OAuth)
- Runner: `src/query_runner.py` (POST /backend/index.php/runQuery)
- Queries existentes: `queries/vendas/`, `queries/sangrias/`

## Notes
- NRORG = 5292 é o Grupo do Nô. Sempre filtrar.
- QueryRunner tem limitação: SUM com certas colunas (VRMOVIMENTO) pode retornar 0 rows. Usar COUNT + GROUP BY funciona. 
- Sessão cacheada em `session_bi.json` (4h TTL)
- Colunas Oracle são UPPERCASE. Scripts de carga fazem lowercase.
