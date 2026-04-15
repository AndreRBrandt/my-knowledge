---
type: domain
status: living
project: [capra-dbt]
tags: [dbt, rh, pontomais, people-analytics, domain]
created: 2026-04-15
related: "[[dbt Architecture]], [[capra-dbt]]"
---

# RH (dbt)

> Domain de People Analytics — dados de jornada, banco de horas, escalas, folha.

## Modelos

| Modelo | Tipo | Grão | Uso |
|--------|------|------|-----|
| `obt_rh_diario` | OBT | Funcionário + dia | KPIs RH, banco de horas |

## Intermediate

`intermediate/rh/` — lógica de cálculo de banco de horas, tratamento de abonos, etc.

## Status

Domain **ATIVO em expansão.** Hoje tem apenas 1 modelo. Refatoração completa da ingestão do PontoMais é **pendente** — ver `_legacy_reference/rh_pontomais_gold_layer/` para a estrutura antiga (que rodava em BIMachine) como referência.

## Fonte

### PontoMais (API v2)
- 35 endpoints (20 instantâneos + 15 relacionais)
- Ingestão: a refatorar (hoje usa scripts PowerShell legados)
- Staging: `staging/pontomais/` (subfolder)

## Conceitos

### Banco de horas
Saldo positivo/negativo entre horas trabalhadas e horas previstas pela escala. Regra: cálculo D-1 (dados de ontem, não tempo real).

### Jornada
Turno/escala do funcionário (6x1, 5x2, 4h estagiário, etc).

### Abonos
Justificativas de ausência: atestado médico, férias, licença, falta justificada.

## Relações cross-domain

- **→ Core:** dim_filial
- **→ Vendas:** correlação operador/vendedor com desempenho (futuro)
- **→ Financeiro:** folha de pagamento ↔ fluxo (futuro)

## Legacy

⚠️ `_legacy_reference/rh_pontomais_gold_layer/` contém a estrutura antiga (BIMachine). **Não usar**, consultar apenas como referência de regras de negócio + mapeamento da API v2.
