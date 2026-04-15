---
type: concept
status: stable
domain: business-rules
project: [capra-dbt, capra-analytics, bode-api]
tags: [domain, business-rule, faturamento, calculation, critical]
created: 2026-02-27
related: "[[Vendas (dbt)]], [[Glossary (consolidated)]]"
---

# Faturamento Calculation

> **Regra crítica:** `Faturamento = SUM(vr_total_item)`. NUNCA somar gorjeta separadamente — `vr_total_item` JÁ a inclui.

## Por que existe esta regra
Bug recorrente: somar `vr_total_item + vr_taxa_servico` (ou somar `vr_acrescimo_item` separadamente) duplica a gorjeta no faturamento.

| Fórmula | Resultado | Status |
|---------|-----------|--------|
| `SUM(vr_total_item)` | 152.278 | ✅ CORRETO |
| `SUM(vr_total_item) + SUM(vr_taxa_servico)` | 163.342 | ❌ ERRADO (gorjeta 2x) |

## Por que `vr_total_item` já inclui gorjeta
No Teknisa:
```
vr_total_item = (qty * preco) - desconto_item + acrescimo_item
```

`vr_acrescimo_item` = gorjeta (taxa de serviço) **rateada por item**. Não é nem `acrescimo` nem `taxa` separada — é a gorjeta distribuída no nível do item.

## Onde é aplicado
| Camada | Local |
|--------|-------|
| dbt staging | `staging.stg_vendas_itens` (preserva `vr_total_item`) |
| dbt gold | `obt_vendas_item.valor_liquido` (alias canônico do faturamento) |
| Frontend | `useKpis`, `useChart`, `useLojas`, `useVendedores`, `useProdutos`, `useKpiTrend`, `calcFaturamento()` |
| Schemas | Measure `valorLiquido` mapeia para `valor_liquido` na obt |

## Edge cases
- Cancelamentos têm `vr_total_item` próprio (negativo após reversão); somar diretamente já desconta corretamente
- Venda zerada (`vr_total_item = 0`) não deve ser excluída — pode representar combo gratuito ou cortesia, conta como "venda atendida"

## Onde NÃO usar
Esta regra é específica do **faturamento bruto líquido** vendido. Não usar como base para:
- **Margem** (precisa subtrair CMV separadamente)
- **Receita reconhecida fiscal** (regras de competência distintas)
- **Custo da gorjeta para garçom** (precisa de `vr_acrescimo_item` isolado, NÃO de `vr_total_item`)

## Validação
Toda mudança em queries de faturamento DEVE validar contra o número canônico (152.278 no fixture de teste). Se divergir, alguém somou gorjeta duas vezes.
