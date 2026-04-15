---
type: concept
status: stable
project: [capra-ui]
tags: [architecture, capra-ui, measures, calculations, transforms]
created: 2025-02-05
related: "[[ADR-UI-006 Measures and Transforms]], [[Schema Builder]]"
---

# Measure Engine

> Pipeline declarativo que processa dados brutos aplicando calculators, transforms e formatters definidos no schema.

## O que é
Engine que recebe um `SchemaDefinition` + `rawData[]`, retorna `ProcessedData[]` enriquecido com cálculos e formatações. Schema descreve "o que"; engine faz "como".

## Por que existe
Cálculos comuns (variação %, participação %, ratios, formatação) estavam duplicados em 50+ lugares. Fórmulas divergiam entre composables. Manutenção exigia mudar em N arquivos. Centralizar = DRY, consistência, testabilidade.

## Como funciona

### Composição do pipeline
1. **Calculators** geram novas medidas a partir das brutas:
   - `variation` — `((atual - anterior) / anterior) * 100`
   - `participation` — `(valor / total) * 100`
   - `ratio` — `numerator / denominator`
   - `average`, `movingAverage`, `cumulative`, `ranking`, `intensity`
2. **Transforms** anotam cada linha com campos derivados:
   - `addFormatted` — strings prontas para display
   - `addTrend` — classe CSS (`trend--positive`/`negative`) e ícone
   - `addRanking` — posição ordenada
   - `addPreviousPeriod` — coluna do período anterior
3. **Formatters** aplicam locale (`pt-BR`) e currency (`BRL`): `currency`, `percent`, `number`, `duration`.

### Schema declara
```typescript
calculatedMeasures: {
  ticketMedio:   { type: 'ratio', numerator: 'faturamento', denominator: 'vendas', format: 'currency' },
  variacaoFat:   { type: 'variation', measure: 'faturamento' },
  participacao:  { type: 'participation', measure: 'faturamento', total: 'faturamentoTotal' },
}
transforms: { addFormatted: true, addTrend: { measures: ['faturamento'] } }
```

### Engine enriquece linha
Input `{ name: "Loja A", faturamento: 150000, faturamentoAnterior: 120000 }` →
Output adiciona `faturamentoFormatted`, `faturamentoVariacao`, `faturamentoVariacaoFormatted`, `faturamentoTrendClass`, `faturamentoTrendIcon`, `participacaoLoja`, `ranking`.

## Relações
- `[[ADR-UI-006 Measures and Transforms]]` — decisão
- `[[Schema Builder]]` — produz o `SchemaDefinition` consumido
- `[[Generic Composables Layer]]` — `useAnalyticData` orquestra QueryManager + MeasureEngine

## Extensão
`engine.registerCalculator(type, fn)` / `registerTransformer` / `registerFormatter` permite plug-in.
