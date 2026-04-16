---
type: meta
status: stable
created: 2026-04-16
---

# Guia: Scripts de Extração de Dados

> Padrão genérico para criar scripts que extraem dados de fontes externas, transformam e carregam em destino. Serve como referência pra IA montar novos scripts seguindo o mesmo modelo.

## Principios

1. **Script = função pura**: recebe parâmetros, retorna dados. Sem side effects ocultos.
2. **Credenciais via env**: nunca hardcoded. Sempre `.env` ou variáveis de ambiente.
3. **Idempotente**: rodar 2x não duplica dados (upsert ou delete+insert).
4. **Validável**: output esperado documentado. Contagens, amostras, checksums.
5. **Incremental**: preferir carga por período (dia a dia) a full snapshot pra tabelas grandes.

## Anatomia de um Script de Extração

```
┌──────────────────────────────────────────────┐
│  1. AUTENTICAÇÃO                              │
│     - Login na fonte (OAuth, API key, etc)    │
│     - Sessão com TTL (reuso, não login/query) │
├──────────────────────────────────────────────┤
│  2. EXTRAÇÃO                                  │
│     - Query parametrizada (SQL, REST, GraphQL)│
│     - Paginação ou iteração por data          │
│     - Timeout + retry com backoff             │
├──────────────────────────────────────────────┤
│  3. TRANSFORMAÇÃO (mínima)                    │
│     - Lowercase keys                          │
│     - Normalizar datas (ISO YYYY-MM-DD)       │
│     - Normalizar nulls                        │
│     - Garantir mesmo set de colunas por row   │
├──────────────────────────────────────────────┤
│  4. CARGA                                     │
│     - Batch upsert (500 rows default)         │
│     - Schema + tabela + on_conflict explícito │
│     - Log de progresso (rows/batch)           │
├──────────────────────────────────────────────┤
│  5. VALIDAÇÃO                                 │
│     - Count source vs destination             │
│     - Checksum (SUM de coluna numérica)       │
│     - Sample visual (primeiros 5 rows)        │
└──────────────────────────────────────────────┘
```

## Pattern: QueryRunner (SQL direto em banco remoto)

Quando a fonte é um banco relacional acessível via API intermediária (ex: Teknisa QueryRunner).

### Estrutura do script

```python
"""
{Descrição}: {Fonte} → {Destino}
Tables: {lista de tabelas}
Usage: python script.py [parametros]
Env: {variáveis necessárias}
"""

# 1. Auth
from fonte.auth import get_session
from fonte.query_runner import run_query

# 2. Extract
sql = "SELECT * FROM {TABELA} WHERE {FILTRO}"
rows = run_query(sql)

# 3. Transform
clean_rows = normalize_keys(rows)  # lowercase, uniform columns

# 4. Load
upsert_to_destination(table, clean_rows, on_conflict="pk1,pk2")

# 5. Validate
print(f"Source: {len(rows)}, Destination: {count_destination()}")
```

### Parâmetros obrigatórios por tabela

Cada tabela precisa definir:

| Parâmetro | Exemplo | Por quê |
|---|---|---|
| `table` | "movcaixa" | Nome na fonte e destino |
| `pk` | "cdfilial,cdcaixa,nrseqvenda,nrsequmovi" | Para upsert sem duplicatas |
| `date_column` | "DTMOVIMCAIXA" | Para extração incremental |
| `date_format` | `TO_CHAR(col, 'YYYY-MM-DD')` | Normalização de data |
| `nrorg` | 5292 | Filtro de organização (Teknisa) |
| `schema` | "raw" | Schema destino no Supabase |

## Pattern: API Webtoken (REST GET com token)

Quando a fonte é uma API REST que retorna JSON.

```python
# 1. Auth — token fixo (configurado no provider)
headers = {"Webtoken": env.WEBTOKEN}

# 2. Extract
response = requests.get(url, headers=headers, timeout=90)
rows = response.json()

# 3-5. Transform + Load + Validate (mesmo padrão)
```

## Pattern: Carga Incremental (dia a dia)

Para tabelas transacionais grandes (>100K rows).

```python
for date in date_range(start, end):
    # Extract do dia
    rows = run_query(f"... WHERE TO_CHAR(dt_col, 'YYYY-MM-DD') = '{date}'")
    
    if not rows:
        continue
    
    # Delete + Insert (evita conflitos de PK complexa)
    delete_by_date(table, date)
    insert_batch(table, rows)
    
    print(f"  {date}: {len(rows)} rows")
```

**Quando usar dia a dia vs bulk:**
- < 10K rows/dia → dia a dia (seguro, idempotente)
- Dimensões (< 1K total) → bulk snapshot (delete all + insert)
- > 50K rows/dia → considerar pagination na query

## Como a IA cria um novo script

Ao receber um pedido como "preciso carregar dados de X pra Y":

1. **Consultar vault**: ler `[[Teknisa Oracle Data Map]]` pra PK, DATA_REF, relacionamentos
2. **Escolher pattern**: QueryRunner (SQL direto) ou Webtoken (REST API)
3. **Definir parâmetros**: tabela, PK, date_column, schema destino
4. **Montar script** seguindo a anatomia acima
5. **Registrar** em `80-Scripts/operations/` com o template `[[Script]]`
6. **Validar**: rodar com --dry-run ou 1 dia antes de carga completa

### Checklist antes de rodar

- [ ] Credenciais no `.env` (nunca hardcoded)
- [ ] PK correta (consultar `[[Teknisa Oracle Data Map]]`)
- [ ] DATA_REF correta (consultar mapeamento — NUNCA usar DTULTATU pra transacionais)
- [ ] Schema destino correto (`raw` pra dados brutos)
- [ ] Teste com 1 dia antes de carga full
- [ ] Contagem source vs destination pós-carga

## Referências

- `[[Teknisa Oracle Data Map]]` — tabelas, PKs, volumes, relacionamentos
- `[[Data Pipeline Status]]` — estado atual de cada tabela
- `[[Script]]` template em 70-Templates/
- Scripts existentes:
  - `[[teknisa-query-runner]]` — consulta Oracle direta
  - `[[teknisa-load-supabase]]` — carga QueryRunner → Supabase
  - `[[teknisa-sync-trigger]]` — trigger sync automático
  - `[[teknisa-fetch-csv]]` — download API → CSV
