---
type: reference
status: living
domain: glossary
tags: [domain, glossary, vocabulary]
created: 2026-04-15
related: "[[Filial]], [[Empresa]], [[Roles]], [[Permissions]]"
---

# Glossary (consolidated)

> Termos cross-project, com link para nota dedicada quando existe.

## Organização

| Termo | Definição | Nota dedicada |
|-------|-----------|---------------|
| **Filial** | Unidade operacional individual (loja física). Particiona dados e permissões. | `[[Filial]]` |
| **Loja** | Sinônimo de Filial (uso comum no domínio operacional). | — |
| **Unidade** | Sinônimo de Filial (nome do cubo Auditoria no BIMachine — `[[Filter Registry]]`). | — |
| **Empresa / Marca** | Agrupamento por CNPJ/marca (Bode do Nô, Burguer do Nô). Não tem coluna explícita ainda. | `[[Empresa]]` |

## RBAC

| Termo | Definição | Nota dedicada |
|-------|-----------|---------------|
| **Role** | Papel do usuário (`adm`, `diretor`, `gestor_filial`). Define permissões default. | `[[Roles]]` |
| **Permission** | Capacidade do sistema (`view_revenue`, `export_data`). Tem N valores possíveis. | `[[Permissions]]` |
| **Permission value** | Valor da permissão (`absolute`, `percentage`, `hidden`). Cada valor tem um `level` (maior = mais restritivo). | `[[Permissions]]` |
| **User override** | Sobrescreve permissão default da role para um usuário, sempre mais restritivo. | `[[User Permissions Override]]` |
| **AuthUser** | Struct Rust em `bode-api` com `id, email, role, filiais, permissions: HashMap`. | `[[Auth Flow JWKS]]` |

## Pipeline de dados

| Termo | Definição |
|-------|-----------|
| **Raw** | Dados crus extraídos das fontes (Teknisa, Pontomais, etc.). Schema: `raw`. |
| **Staging** | Limpeza inicial (renames, types, NULL handling). Schema: `staging`. |
| **Intermediate** | Transformações intermediárias (joins, derivações). Schema: `intermediate`. |
| **Marts (Gold)** | Tabelas finais consumidas por API/BI. Schema: `gold`. Equivalente a "Gold layer" em medallion. |
| **OBT** | One Big Table — tabelas wide consolidadas em gold (`obt_vendas_item`, `obt_cancelamentos`, `obt_financeiro`). |
| **Refresh incremental** | `refresh_gold_incremental.sql` — INSERT de novas linhas em obts (sem rebuild total). |

## Métricas e cálculos

| Termo | Definição |
|-------|-----------|
| **Faturamento** | `SUM(vr_total_item)` — JÁ inclui gorjeta rateada via `vr_acrescimo_item`. NUNCA somar gorjeta separadamente. Ver `[[Faturamento Calculation]]`. |
| **CMV** | Custo da Mercadoria Vendida. Ver dbt domain. |
| **Ticket médio** | `Faturamento / Vendas` (calcMeasure tipo `ratio`). |
| **YoY** | Year-over-Year — comparativo principal (mesmo período do ano anterior). Ver `[[Comparison Layers (3-tier)]]`. |
| **MA4** | Média Móvel 4 semanas (mesmo dia da semana). Só para período de dia único. |
| **Variação** | `((atual - anterior) / anterior) * 100`. |
| **Participação** | `(valor / total) * 100`. |

## Stack e arquitetura

| Termo | Definição |
|-------|-----------|
| **bode-api** | Backend Rust (Axum + sqlx). Substitui bode-analytics-api via Strangler Fig. Ver `[[bode-api]]`. |
| **bode-analytics-api** | Backend TS legado (Hono/Node). Em decommission. |
| **capra-ui** | Framework Vue (`@capra-ui/core`). Public. Ver `[[capra-ui]]`. |
| **capra-analytics** | App de dashboards consumindo capra-ui. |
| **capra-dbt** | Projeto dbt (raw → staging → intermediate → marts em Postgres). Ver `[[capra-dbt]]`. |
| **Adapter** | Camada de tradução entre UI e fonte de dados. Ver `[[Adapter Pattern (UI Layer)]]`. |
| **MDX** | Linguagem OLAP do BIMachine. Em decommission junto com BIMachine (ver `[[GADR-008 BIMachine Deprecation]]`). |

## BIMachine (legacy — deprecated)

| Termo | Definição |
|-------|-----------|
| **BIMachine** | Plataforma BI proprietária. Oficialmente obsoleto. Ver `[[GADR-008 BIMachine Deprecation]]`. |
| **cockpitContext** | API JS do BIMachine para widgets. Vai sumir junto. |
| **ReduxStore** | Variável global do BIMachine (`window.ReduxStore`). |
| **BIMACHINE_FILTERS** | Globals do BIMachine (acessada via `getBIMachineFilters()` wrapper). |

## Workflow

| Termo | Definição |
|-------|-----------|
| **Spec-driven** | Spec descreve O QUE e POR QUE; código é do implementador (André). Ver `[[GADR-004 Spec-Driven Workflow with Consultant LLM]]`. |
| **Trunk-based** | Feature → main (squash) → tag → prod. Ver `[[GADR-002 Trunk-Based Development]]`. |
| **Strangler Fig** | Substituição incremental do legado. Ver `[[Strangler Fig Migration]]`. |
| **Framework-First** | App nunca sobrescreve visual do framework. Ver `[[Framework-First Visual Ownership]]`. |
