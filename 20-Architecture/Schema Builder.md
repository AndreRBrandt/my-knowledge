---
type: concept
status: stable
project: [capra-ui]
tags: [architecture, capra-ui, schema, dx, builder-pattern]
created: 2025-02-05
related: "[[ADR-UI-005 Schema Builder]], [[Measure Engine]], [[Filter Registry]]"
---

# Schema Builder

> Builder fluent + Registry singleton + Factories com defaults inteligentes para definir schemas analíticos sem boilerplate.

## O que é
Combinação de três peças:
- **SchemaBuilder** — API fluent que monta um `SchemaDefinition` por chaining
- **SchemaRegistry** — singleton onde schemas se registram para lookup global
- **Factories** — `DimensionFactory`, `MeasureFactory`, `FilterConfigFactory` aplicam defaults (ex: dimensão `loja` gera `hierarchy: "[loja].[Todos].Children"` automaticamente)

## Por que existe
Schemas anteriores tinham 80% de estrutura idêntica replicada (4 arquivos), padrões repetitivos copy-pasted, sem validação compile-time. Adicionar schema novo = copiar centenas de linhas. Builder + factories eliminam o boilerplate.

## Como funciona
```typescript
const schema = new SchemaBuilder()
  .setDataSource('TeknisaVendas', 'teknisa-vendas')
  .addDimension('loja')                                   // hierarchy/dimension auto-derivados
  .addDimension('data', { type: 'time' })
  .addMeasure('valorLiquido', { label: 'Faturamento', format: 'currency' })
  .addCategory('DELIVERY', 'Delivery', '#F97316')
  .addFilterConfig('kpiFaturamento', { accepts: ['data', 'loja'] })
  .build()

schemaRegistry.register(schema)
const measure = schemaRegistry.getMeasure('teknisa-vendas', 'valorLiquido')
```

Validação ocorre **durante** o `build()` — erros aparecem antes de runtime.

## Relações
- `[[ADR-UI-005 Schema Builder]]` — decisão
- `[[Measure Engine]]` — consome `SchemaDefinition` para enriquecer dados
- `[[Filter Registry]]` — referencia dimensões definidas no schema
- `[[Dimension Discovery]]` — preenche membros de dimensão dinamicamente
- `[[ADR-UI-017 Endpoint Schema Discovery]]` — gera código do builder a partir de endpoint

## Anti-padrão
JSON Schema ou config estática perde type-safety, IDE completion, e validação em build-time.
