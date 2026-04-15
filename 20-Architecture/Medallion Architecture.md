---
type: architecture
status: living
project: []
tags: [data, medallion, pipeline]
created: 2026-04-15
related: "[[dbt Architecture]], [[Data Flow]]"
---

# Medallion Architecture

> Padrão de arquitetura de dados em **camadas progressivas** (bronze/raw → silver/staging → gold/marts), cada uma com propósito específico. É o backbone conceitual do pipeline de dados do Capra.

## O que é

Medallion é um padrão onde dados passam por **camadas hierárquicas de refinamento**, cada uma com garantias crescentes de qualidade e valor de negócio:

```
BRONZE (raw)      → dados brutos, exatamente como chegam da fonte
      ↓ limpeza
SILVER (staging)  → tipados, deduplicados, nulls tratados
      ↓ transformação
GOLD (marts)      → pronto pra consumo de negócio, joins feitos, agregações
```

## Por que existe

- **Separação de responsabilidades:** cada camada tem um papel claro, ninguém faz tudo
- **Rastreabilidade:** se um número está errado no gold, dá pra descer até o raw e investigar
- **Reprocessamento:** se uma regra muda, reprocessa só a partir da camada afetada
- **Contratos:** gold tem schema estável que API consome; staging pode mudar conforme fonte evolui
- **Debug:** problemas se isolam na camada onde ocorrem

## Como se aplica no Capra

Ver `[[dbt Architecture]]` pra detalhes de implementação. Resumo:

| Nosso nome | Termo genérico | Schema | Materialização |
|------------|---------------|--------|----------------|
| Raw | Bronze | `raw` (Postgres) | tables (ingestão) |
| Staging | Silver (inicial) | `staging` | views |
| Intermediate | Silver (final) | `staging` | views |
| Marts | Gold | `gold` | tables |

**Divergência nominal:** muitos projetos usam "silver" pra camada única entre raw e gold. Aqui decompomos silver em **staging** (limpeza) + **intermediate** (lógica de negócio reutilizável).

## Regras por camada

### Raw (Bronze)
- **O que vai:** dados exatamente como chegam da API/crawler
- **Transformações:** nenhuma (exceto append/upsert)
- **Quem escreve:** ingestion scripts
- **Quem lê:** apenas camada staging
- **Ciclo:** append incremental (não deletar histórico)

### Staging
- **O que vai:** tipagem correta, rename de colunas, nulls tratados, dedup
- **Transformações:** superficiais e mecânicas (não lógica de negócio)
- **Quem escreve:** dbt (models `stg_*`)
- **Quem lê:** intermediate + marts
- **Regra:** 1 model = 1 tabela raw (com raras exceções)

### Intermediate
- **O que vai:** joins, cálculos reutilizáveis, lógica compartilhada
- **Transformações:** lógica de negócio não-final
- **Quem escreve:** dbt (models `int_*`)
- **Quem lê:** marts
- **Regra:** se um cálculo é usado em >1 mart, sobe pra intermediate

### Marts (Gold)
- **O que vai:** dados prontos pra consumo de negócio
- **Transformações:** agregações finais, estruturas otimizadas pra query
- **Quem escreve:** dbt (models `fct_*`, `dim_*`, `obt_*`)
- **Quem lê:** API, analytics, relatórios
- **Regra:** schema é contrato com o consumidor — mudança quebra API/frontend

## Anti-patterns

- ❌ **Joins em staging** — staging deve ser 1:1 com raw na maioria dos casos
- ❌ **Lógica de negócio em staging** — sobe pra intermediate
- ❌ **Pular camada** — mart não deve ler direto de raw
- ❌ **Schema volátil em marts** — quebra consumers silenciosamente
- ❌ **Misturar grains** — fato de venda diária não mistura com fato de venda por item

## Relações

- `[[dbt Architecture]]` — implementação concreta no Capra
- `[[GADR-002 Silver Gold Separation]]` — decisão canônica sobre a separação
- `[[Data Flow]]` — mapa end-to-end do pipeline

## Referências externas

- [Databricks Medallion](https://www.databricks.com/glossary/medallion-architecture) — origem do termo
- [dbt Best Practices — layers](https://docs.getdbt.com/best-practices/how-we-structure/1-guide-overview)
