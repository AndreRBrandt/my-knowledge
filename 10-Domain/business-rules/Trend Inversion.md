---
type: concept
status: stable
domain: business-rules
project: [capra-analytics, capra-ui]
tags: [domain, business-rule, kpi, ux]
created: 2025-02-15
related: "[[Comparison Layers (3-tier)]], [[Measure Engine]]"
---

# Trend Inversion

> Algumas métricas têm interpretação invertida: queda = bom (custos, reclamações, devoluções). Sinalizar via `invertTrend: true` no schema/KPI para que cor/ícone reflitam corretamente.

## Conceito
Por default, KPIs interpretam:
- Aumento → positivo (verde, ↑)
- Queda → negativo (vermelho, ↓)

Métricas invertidas:
- Aumento → negativo (vermelho, ↑)
- Queda → positivo (verde, ↓)

## Quando aplicar
| Métrica | Direção positiva |
|---------|------------------|
| Faturamento, vendas, ticket médio | aumento |
| Margem, lucratividade | aumento |
| **CMV, custo, despesas** | **queda** |
| **Reclamações, cancelamentos, devoluções** | **queda** |
| **Tempo médio de espera** | **queda** |

## Implementação

### Schema declarativo
```typescript
calculatedMeasures: {
  variacaoCMV: {
    type: 'variation',
    measure: 'cmv',
    invertTrend: true,   // ← marca a métrica como invertida
  }
}
```

### Lógica do MeasureEngine
```typescript
const trend = ((current - previous) / previous) * 100
const isPositive = invertTrend ? trend < 0 : trend > 0

// trendClass:    isPositive ? 'trend--positive' : 'trend--negative'
// trendIcon:     isPositive ? TrendingUp : TrendingDown   (mas o sentido inverte em invertTrend)
```

## UX em invertTrend
Mesmo invertido, o ÍCONE acompanha o número (não a interpretação). Um custo que SOBE 25% mostra `↑ +25%` em vermelho — usuário lê "subiu, é ruim". A cor sinaliza a interpretação; o ícone, a direção real do número.

## Onde é usado
- `[[Measure Engine]]` (capra-ui) — calculator `variation` lê `invertTrend`
- KpiCard prop `invertTrend: boolean`
- Schemas que declaram CMV, cancelamentos, custo, etc.
