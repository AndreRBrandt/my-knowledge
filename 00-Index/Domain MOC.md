---
type: moc
status: living
created: 2026-04-15
---

# Domain MOC

> Conhecimento de negócio **agnóstico de projeto**. Conceitos, entidades, regras de domínio.

## Data Domains (marts dbt)
Cada domain corresponde a um mart em `capra-dbt/models/marts/`:

- `[[Core (dbt)]]` — dimensões compartilhadas (dim_filial)
- `[[Vendas (dbt)]]` — transações, itens, cancelamentos
- `[[Financeiro (dbt)]]` — AP, AR, fluxo, caixa
- `[[CMV (dbt)]]` — custo mercadoria vendida (ficha técnica)
- `[[Avaliacoes (dbt)]]` — feedback clientes (Avalyo, Wifire)
- `[[RH (dbt)]]` — people analytics (banco de horas, escalas)
- `[[Estoque (dbt)]]` — **não implementado ainda**

## Foodservice (a migrar)
- Filial (9 filiais, códigos 0001-0009)
- Unidades (BDN, Burguer, Italiano)
- PDV, Turnos

## Permissions (a migrar do bode-api)
- RBAC Model
- Role, Permission

## Glossário
- _(a criar: consolidar GLOSSARIO_GLOBAL + específicos)_

---

Template: `[[Concept]]`
