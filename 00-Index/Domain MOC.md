---
type: moc
status: living
created: 2026-04-15
---

# Domain MOC

> Conhecimento de negócio **agnóstico de projeto**. Conceitos, entidades, regras de domínio.

## Organização
- `[[Filial]]` — unidade operacional (loja física)
- `[[Empresa]]` — agrupamento por marca/CNPJ (Bode do Nô / Burguer do Nô)

## RBAC (controle de acesso)
- `[[Roles]]` — papéis (`adm`, `diretor`, `gestor_filial`)
- `[[Permissions]]` — catálogo + valores com `level` (maior = mais restritivo)
- `[[User Permissions Override]]` — sobrescrita restritiva por usuário
- `[[RBAC Cascade Model]]` — modelo mental completo (JWT → AuthUser)

## Business Rules
- `[[Faturamento Calculation]]` — `SUM(vr_total_item)`, NUNCA somar gorjeta separadamente
- `[[Comparison Layers (3-tier)]]` — YoY + Short-term + MA4
- `[[Trend Inversion]]` — métricas onde queda = bom (custos, reclamações)

## Data Sources & Pipeline
- `[[Teknisa Oracle Data Map]]` — mapeamento completo: tabelas, PKs, volumes, JOINs, scripts de acesso
- `[[Data Pipeline Status]]` — estado atual de cada tabela (fresh, stale, gaps, acoes pendentes)

## Data Domains (marts dbt)
Cada domain corresponde a um mart em `capra-dbt/models/marts/`:

- `[[Core (dbt)]]` — dimensões compartilhadas (dim_filial, dim_produto, dim_operador)
- `[[Vendas (dbt)]]` — transações, itens, cancelamentos, vendas por hora, delivery
- `[[Financeiro (dbt)]]` — AP, AR, fluxo, caixa, sangrias, fechamento de caixa
- `[[CMV (dbt)]]` — custo mercadoria vendida (ficha técnica)
- `[[Avaliacoes (dbt)]]` — feedback clientes (Avalyo, Wifire)
- `[[RH (dbt)]]` — people analytics (banco de horas, escalas)
- `[[Estoque (dbt)]]` — **não implementado ainda**

## Glossário
- `[[Glossary (consolidated)]]` — vocabulário cross-project (organização, RBAC, pipeline, métricas, stack, BIMachine legacy, workflow)

---

Template: `[[Concept]]`
