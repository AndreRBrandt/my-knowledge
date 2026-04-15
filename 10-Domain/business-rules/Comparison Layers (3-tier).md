---
type: concept
status: stable
domain: business-rules
project: [capra-analytics, capra-dbt]
tags: [domain, business-rule, comparison, kpi, time-series]
created: 2026-03-04
related: "[[Vendas (dbt)]], [[Faturamento Calculation]]"
---

# Comparison Layers (3-tier)

> Sistema de 3 tipos de comparação que contextualizam métricas: YoY (sazonalidade), Short-Term (tendência recente), MA4 (suavização de outliers).

## Por que três camadas
Uma única comparação engana. YoY mostra sazonalidade mas perde mudanças recentes. Short-term mostra tendência mas é vulnerável a outliers. MA4 suaviza, mas requer série de dias úteis comparáveis.

## 1. YoY (Year-over-Year) — Comparativo Principal
Compara com o **mesmo período do ano anterior**. Reflete sazonalidade real (Carnaval, Natal, alta/baixa temporada).

- **Exibido em:** KPI cards (seta + %), DataTable trend badge principal
- **Config:** `comparison: { type: 'period', dimension: 'DATA_REF', offset: 1, unit: 'year' }`

## 2. Short-Term — Comparativo de Curto Prazo
Comparação anterior próxima. Mostra tendência recente (semana, mês).

- **Exibido em:** Apenas detail views (colunas Δ Anterior)
- **Resolução automática por período:**
  - `lastday` / `today` → 7 dias atrás
  - `thismonth` → mês anterior
  - `thisyear` → ano anterior

## 3. MA4 — Média Móvel 4 Semanas
Média das **últimas 4 ocorrências do mesmo dia da semana**. Suaviza outliers (uma terça anômala não distorce).

- **Restrição:** Só para períodos de **dia único** (lastday, today). Range não aplica.
- **Servidor:** busca `IN(d-7, d-14, d-21, d-28)`, agrega, divide por 4
- **Config:** `comparison: { type: 'moving_average', dimension: 'DATA_REF', count: 4 }`

## Regra de exibição
| Período selecionado | YoY | Short-term | MA4 |
|--------------------|-----|-----------|-----|
| `lastday` / `today` | ✅ | ✅ (7 dias atrás) | ✅ |
| `thismonth` | ✅ | ✅ (mês anterior) | ❌ |
| `thisyear` | ✅ | ✅ (ano anterior) | ❌ |
| `last7days` / range | ✅ | ❌ | ❌ |

## Implementação
- Query MDX/SQL com `ParallelPeriod` para YoY
- Helper centralizado em `usePeriodHelper.ts` (`buildKpiQueryWithPeriod`, `buildTableQueryWithPeriod`)
- Ver `[[ADR-UI-014 App Query Conventions]]`

## Onde é usado
- KPI cards de vendas, faturamento, ticket médio
- DataTables de detalhe (lojas, produtos, vendedores)
- Charts de tendência

## Anti-pattern
Usar Short-term como métrica principal — perde sazonalidade. YoY é o headliner; short-term é contexto secundário.
