---
type: concept
status: proposed
tags: [architecture, etl, data-extraction, rust, bode-sync, metadata-driven]
created: 2026-04-15
related: "[[teknisa-crawler]], [[Teknisa BI Auth Flow]], [[Crate Boundaries]], [[bode-api Phase 2 Deploy Plan]]"
---

# Data Extraction Engine

> Engine de extração de dados genérica e configurável em Rust, vivendo no crate `bode-sync` do bode-api. Generaliza o `[[teknisa-crawler]]` Python existente. Primeiro conector é Teknisa BI; o design suporta adicionar novos conectores (Teknisa Retail, IFood, SugestCard, generic Postgres, CSV upload) sem mudar o core.

## Por que existe

1. **Crawler Python é primitivo, frágil, monolítico** — auth + queries hardcoded por script, sem retries, sem logging estruturado, sem testes
2. **Toda nova fonte requer reescrever auth + execução** — código duplicado entre Teknisa, IFood, etc.
3. **Princípio metadata-driven (D9 / ADR-017):** queries e conectores precisam ser entidades de primeira classe — não código hardcoded
4. **Eixo 3 da migração (produto low-code futuro):** quando admin builder existir, ele expõe esta engine — usuário não-técnico configura nova fonte via UI
5. **CLI standalone:** ferramenta usável fora do contexto bode-api (cron, terminal, outros projetos do workspace)

## Princípios de design

| Princípio | Como se manifesta |
|---|---|
| **Connector como trait** | `trait Connector` em `bode-core` (zero I/O), impls em `bode-sync` |
| **Query como dado, não código** | `Query` é struct serializável: SQL/payload + metadata. Vive em arquivos `.sql` + `.meta.yaml`, não embeded em Rust |
| **Output desacoplado** | Mesmo `Rows` pode ser escrito em Postgres, JSON, CSV — escolhido em runtime |
| **Auth abstraída** | Cada conector implementa sua própria auth (Zeedhi pro Teknisa, OAuth pro IFood, etc.) — engine não conhece o specific |
| **Logging estruturado** | Tracing spans por extração; alimenta Loki/Grafana (Epic J) |
| **Testável contra ground truth** | Sample CSVs do Python crawler viram regression fixtures Rust |

## Arquitetura

```
┌─────────────────────────────────────────────────────────────────┐
│                          bode-core                               │
│  (zero I/O — apenas tipos e traits)                              │
│                                                                  │
│   trait Connector {                                              │
│     async fn authenticate(&self) -> Result<Session>;             │
│     async fn run_query(&self, q: &Query) -> Result<Rows>;        │
│     fn name(&self) -> &'static str;                              │
│   }                                                              │
│                                                                  │
│   struct Query { id, connector, payload, output_schema, ... }    │
│   enum   QueryPayload { Sql(String), ApiCall { ... }, ... }      │
│   struct Rows { columns, data: Vec<HashMap<String, Value>> }     │
│                                                                  │
│   trait Output {                                                 │
│     async fn write(&self, rows: Rows) -> Result<usize>;          │
│   }                                                              │
└──────────────────────────────┬──────────────────────────────────┘
                               │ depends on
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                          bode-sync                               │
│  (impls concretas + binários CLI)                                │
│                                                                  │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│   │ connectors/  │  │   outputs/   │  │    auth/     │          │
│   │ teknisa_bi   │  │ postgres     │  │   zeedhi     │          │
│   │ (futuro:     │  │ json         │  │   (futuro:   │          │
│   │  retail,     │  │ csv          │  │    oauth,    │          │
│   │  ifood,      │  │              │  │    api_key)  │          │
│   │  generic_pg, │  │              │  │              │          │
│   │  csv_file)   │  │              │  │              │          │
│   └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                  │
│   ┌──────────────────────────────────────────────────┐           │
│   │  Extractor (orquestra Connector + Output)        │           │
│   │   .run(query, output) -> ExtractionResult        │           │
│   │   emite tracing span por run                     │           │
│   └──────────────────────────────────────────────────┘           │
│                                                                  │
│   ┌──────────────┐  ┌──────────────┐                             │
│   │ bin/extract  │  │bin/seed_homol│                             │
│   │ CLI genérica │  │ Específico   │                             │
│   └──────────────┘  └──────────────┘                             │
│                                                                  │
│   queries/                                                       │
│   └── teknisa/vendas/                                            │
│       ├── q1_resumo_filial.sql                                   │
│       └── q1_resumo_filial.meta.yaml                             │
│                                                                  │
│   tests/fixtures/                                                │
│   └── teknisa/vendas/                                            │
│       └── q1_resumo_filial.fixture.csv  ← ground truth           │
└──────────────────────────────────────────────────────────────────┘
```

## Boundary com `bode-server` (HTTP API)

A engine **não tem rotas HTTP próprias por enquanto**. O `bode-server` consome a engine via library calls (function calls em Rust). Quando admin builder existir (Eixo 3, fase futura), `bode-server` expõe rotas tipo:
- `POST /api/v2/admin/connectors` (CRUD)
- `POST /api/v2/admin/queries` (CRUD)
- `POST /api/v2/admin/extractions/run?query_id=X` (dispara extração)
- `GET /api/v2/admin/extractions/runs` (history)

Por enquanto: tudo via CLI (`extract`, `seed_homolog`).

## Interação com outros sistemas

| Sistema | Como usa a engine |
|---|---|
| **Staging seed** (`bode_homolog`) | `seed_homolog` CLI roda extrações configuradas → escreve em Postgres |
| **dbt (`capra-dbt`)** | A engine alimenta a camada **raw** que o dbt consome. `extract` salva em tabelas `raw_*`; dbt transforma em `staging_*` → `gold_*` |
| **Telemetria (Epic J)** | Cada execução emite tracing spans (connector, query_id, duration, rows, status) → Loki indexa |
| **Admin builder (Eixo 3, futuro)** | UI de CRUD em cima das entidades `Connector`, `Query`, `Extraction` |

## Conectores planejados

| Conector | Plataforma | Quando | Status |
|---|---|---|---|
| `teknisa_bi` | Teknisa BI (Oracle SQL via Zeedhi) | Phase 2 K | Planejado (port do Python) |
| `teknisa_retail` | Teknisa Retail (relatórios prontos) | Phase 3+ | Não iniciado |
| `ifood_api` | IFood (REST API) | Quando precisar | Não iniciado |
| `sugestcard` | SugestCard | Quando precisar | Não iniciado |
| `generic_postgres` | Qualquer Postgres com JDBC string | Quando precisar | Não iniciado |
| `csv_file` | Upload de arquivo CSV | Para admin builder Eixo 3 | Não iniciado |

## Regras de design

1. **Connector NUNCA conhece Output** (e vice-versa) — Extractor é o único intermediário
2. **Auth fica no Connector**, não em namespace global — cada plataforma tem seu padrão
3. **Query é serializável** — pode vir de arquivo, DB, ou request HTTP
4. **Erros tipados** via `CoreError` — sem `unwrap()`, sem `panic!` em fluxo normal
5. **Sem hardcoded de queries no Rust** — toda query vive em `queries/<connector>/<domain>/`
6. **Credenciais APENAS via env vars** — nunca comitadas, nunca em config files

## Anti-patterns a evitar

- ❌ Connector que escreve em DB de destino — viola separação Connector/Output
- ❌ Output que sabe HTTP do Teknisa — viola encapsulamento
- ❌ Query SQL no código Rust — quebra metadata-driven, dificulta edição não-técnica
- ❌ Lógica de transformação dentro do Connector — extração é pura; transformação é dbt/downstream
- ❌ Hardcoded de credenciais em config files commitados

## Reference

- Implementação atual (Python): `[[teknisa-crawler]]`
- Plano de migração: `[[bode-api Phase 2 Deploy Plan]]` Epic K
- Princípios cross-cutting: `[[ADR-BODE-009 Zero Vendor Lock-in]]`, ADR-BODE-017 Metadata-Driven (a escrever em Phase 2)
