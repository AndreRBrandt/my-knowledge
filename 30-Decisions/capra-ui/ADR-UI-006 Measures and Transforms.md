---
type: decision
status: stable
project: [capra-ui]
tags: [adr, capra-ui, measures, calculations]
created: 2025-02-05
related: "[[Measure Engine]], [[ADR-UI-005 Schema Builder]]"
---

# ADR-UI-006 Measures & Transforms

**Status:** Accepted (implementado 2025-02-05)
**Date:** 2025-02-05

## Context
Cálculos comuns espalhados: variação % (20+ ocorrências), participação % (30+), formatação (50+ em 6+ arquivos), tendência CSS (15+). Resultado: duplicação, inconsistência, manutenção difícil, lógica não testável.

## Decision
**MeasureEngine** processa dados brutos e aplica calculators + transforms + formatters declarativamente, configurados no schema.

Calculators: `variation`, `participation`, `ratio`, `average`, `movingAverage`, `cumulative`, `ranking`, `intensity`.
Transforms: `addFormatted`, `addTrend`, `addRanking`, `addPreviousPeriod`.
Formatters: `currency`, `percent`, `number`, `duration`.

Schema declara `calculatedMeasures` e `transforms`; engine enriquece dados em runtime (ex: `faturamentoVariacao`, `faturamentoTrendClass`, `participacaoLoja`).

Implementação: `src/core/measures/`.

## Consequences

### Positive
- DRY: cálculo definido uma vez
- Mesma definição → mesmo resultado (consistência)
- Calculators isolados, testáveis
- Schema descreve "o que", engine faz "como"

### Negative
- Camada adicional de processamento
- Migração de código existente

## References
- `[[Measure Engine]]` — concept
