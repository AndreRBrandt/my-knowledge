---
type: decision
status: stable
project: [capra-ui]
tags: [adr, capra-ui, schema, dx]
created: 2025-02-05
related: "[[Schema Builder]], [[ADR-UI-006 Measures and Transforms]]"
---

# ADR-UI-005 Schema Builder

**Status:** Accepted (implementado 2025-02-05)
**Date:** 2025-02-05

## Context
Schemas existentes tinham 80% de estrutura idêntica replicada (4 arquivos), padrão repetitivo de dimensões (`hierarchy`/`dimension` sempre derivados do nome), mappings de label/cor separados, sem validação compile-time, difícil criar schema novo (copy/paste de centenas de linhas).

## Decision
**SchemaBuilder fluent + SchemaRegistry singleton + Factories** (`DimensionFactory`, `MeasureFactory`, `FilterConfigFactory`) com defaults inteligentes.

```typescript
const schema = new SchemaBuilder()
  .setDataSource('TeknisaVendas', 'teknisa-vendas')
  .addDimension('loja')        // hierarchy/dimension auto-gerados
  .addMeasure('valorLiquido', { label: 'Faturamento', format: 'currency' })
  .addCategory('DELIVERY', 'Delivery', '#F97316')
  .build()
```

## Consequences

### Positive
- DRY (padrões definidos uma vez)
- Type-safe, IDE completion
- Validação durante construção
- Schema auto-documentado

### Negative
- Migração de schemas existentes
- Curva de aprendizado
- Casos edge podem não caber no builder

## Alternatives Considered

| Alternativa | Por quê rejeitada |
|-------------|-------------------|
| JSON Schema | Perde type-safety, sem IDE completion |
| Decorators TS | Experimental, verbose |
| Manter as-is | Não resolve duplicação |

## References
- `[[Schema Builder]]` — concept
- `[[ADR-UI-017 Endpoint Schema Discovery]]` — extensão futura
