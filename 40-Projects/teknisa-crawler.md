---
type: hub
status: living
project: [teknisa-crawler]
tags: [python, etl, teknisa, crawler, data-extraction]
created: 2026-04-15
related: "[[bode-api]], [[bode-api Phase 2 Deploy Plan]], [[Teknisa BI Auth Flow]], [[Data Extraction Engine]]"
---

# teknisa-crawler

> Crawler Python para extração automatizada de dados das plataformas Teknisa (BI + Retail). Asset estratégico — único caminho atualmente para puxar dados do Teknisa de forma programática.

## Repo
- **Local:** `C:\Users\ANDRE-BI\bi_projects\projects_script\teknisa_crawler` (privado)
- **Versão pública sanitized:** `projects_script\teknisa_crawler_pub`
- **Status:** ativo, última atualização 2026-03-31

## Stack
- Python 3.13
- `requests` para HTTP
- `python-dotenv` para config
- Auth via Zeedhi backend (Teknisa)

## Estrutura
```
teknisa_crawler/
├── main.py                    # Entry point
├── run_query.py               # Runner: executa .sql e salva amostra.csv
├── src/
│   ├── constants.py           # Headers, paths, TTL
│   ├── config.py              # Configs por plataforma via .env
│   ├── auth.py                # Login multi-plataforma + token mgmt
│   ├── query_runner.py        # SQL via QueryRunner (BI)
│   ├── consulta_cupom.py      # Busca venda por QR/chave/nr nota
│   └── endpoint_scanner.py    # Discovery automática de endpoints
├── queries/
│   └── vendas/
│       ├── q1_resumo_filial/  # query.sql + amostra.csv
│       ├── q2_top_produtos/
│       ├── q3_turno/
│       ├── q4_ticket_medio/
│       └── q5_detalhe_itens/
└── docs/
    ├── CHANGELOG.md
    ├── RULES.md
    ├── PREFERENCES.md
    └── context/
        ├── ARCHITECTURE.md
        ├── MAPPINGS.md
        ├── ENDPOINTS_BI.md       # Mapa do BI
        ├── ENDPOINTS_RETAIL.md   # 22 backend URLs do Retail (relatórios prontos)
        ├── DATABASE.md           # Visão do banco Oracle Teknisa
        └── domains/
            └── vendas.md
```

## Plataformas suportadas

### Teknisa BI (Zeedhi novo, Vue SPA)
- Auth: `POST /backend_login/index.php/login` — JSON flat
- Query: `POST /backend/index.php/runQuery` — Oracle SQL via JDBC config
- Resposta: `{ dataset: { queryResult: [...] } }`

### Teknisa Retail (Zeedhi v4.23 legacy, Bower)
- Auth: `POST /backend_login/index.php/login` — FilterData format
- 22 backend URLs descobertas via scanner — relatórios prontos validados
- Ainda não totalmente mapeado em uso (oportunidade)

## Headers de auth (ambos pós-login)
```
OAuth-Token: <TOKEN>
OAuth-Hash: <hash da org, fixo>
OAuth-Project: <product_id>
Cookie: PHPSESSID=<sessao>
```

Detalhes do fluxo: `[[Teknisa BI Auth Flow]]`

## Queries validadas (com sample CSV ground truth)

| Query | Fonte | Uso |
|-------|-------|-----|
| `vendas/q1_resumo_filial` | VENDA + ITEMVENDA + FILIAL | Resumo diário por filial |
| `vendas/q2_top_produtos` | ITEMVENDA + PRODUTO | Top N produtos |
| `vendas/q3_turno` | VENDA + ITEMVENDA | Análise por turno |
| `vendas/q4_ticket_medio` | VENDA + ITEMVENDA | Ticket médio |
| `vendas/q5_detalhe_itens` | ITEMVENDA + VENDA + PRODUTO | Detalhe linha por linha |
| `consulta_cupom` | VENDA + ITEMVENDA + EFD/SAT | Busca por QR / chave 44 dig / nr nota |

## Convenções importantes do crawler
- **Cache de sessão TTL** (`session_bi.json`, `session_retail.json` — gitignored)
- **Credenciais APENAS no `.env`** (gitignored)
- **Cada query tem `query.sql` + `amostra.csv`** — sample CSV é ground truth
- **Faturamento via item, não header** (regra de cálculo crítica — ver `[[Faturamento Calculation]]`)

## Plano de migração para Rust

**Decisão (S237, 2026-04-15):** porta o crawler pra Rust como **engine genérica** dentro do crate `bode-sync` do bode-api. Não é só rewrite — é generalização.

Ver `[[bode-api Phase 2 Deploy Plan]]` Epic K para o plano detalhado:
- K1-K3: Core engine + auth Zeedhi + connector Teknisa BI em Rust
- K5-K6: Migrar 5 queries existentes → fixtures Python viram regression tests Rust
- K7-K8: CLI standalone (`extract`, `seed_homolog`)

**Após Phase 2 K completo:** crawler Python entra em modo manutenção, eventualmente descontinuado quando engine Rust cobrir 100% dos casos de uso (incluindo Retail).

## Architecture concepts derivados deste asset
- `[[Teknisa BI Auth Flow]]` — fluxo de autenticação Zeedhi (canônico)
- `[[Data Extraction Engine]]` — design da generalização em Rust

## NÃO faz (limitações)
- Não escreve em DB destino — só extrai e gera CSV
- Não orquestra schedule (você roda manual ou via cron externo)
- Não tem retries automáticos sofisticados
- Não emite logs estruturados (Epic K9 vai resolver)
- Não tem CLI genérica — cada query é um script
