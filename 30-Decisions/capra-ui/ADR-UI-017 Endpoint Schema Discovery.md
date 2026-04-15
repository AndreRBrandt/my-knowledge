---
type: decision
status: draft
project: [capra-ui, capra-analytics]
tags: [adr, capra-ui, schema, discovery, dx]
created: 2026-02-23
related: "[[ADR-UI-005 Schema Builder]], [[ADR-UI-012 Dimension Discovery]]"
---

# ADR-UI-017 Endpoint-Driven Schema Discovery

**Status:** Proposed
**Date:** 2026-02-23

## Context
Adicionar nova fonte de dados ao capra-analytics requer criar manualmente schema TS, mapear campos, definir filtros/labels/formatos, criar composables específicos. Repetitivo e exige expertise da arquitetura. Objetivo: tornar plug-and-play — dev passa URL com sample JSON, app infere schema.

## Decision
Sistema em duas camadas:

### Camada 1: Framework (capra-ui)
Service `SchemaDiscovery`:
- GET endpoint → amostra JSON
- Infere tipo de cada campo (`dimension`/`measure`/`date`/`unknown`) por:
  - Tipo de dado JS
  - Heurísticas no nome (sufixos `valor`/`total`/`qtd` → measure; `id`/`cod` → dimension; substring `data`/`date` → date)
  - Cardinalidade (poucos valores → categórico)
- Retorna `InferredSchema` com `confidence: number`

### Camada 2: App (capra-analytics)
Painel em `ConfigPage` → aba "Dev":
- Input URL + headers
- Tabela de campos inferidos (tipo, formato, label, confiança)
- Confirmar/corrigir → gera código TS do schema + composable básico
- Persistência em localStorage

Geração final usa `SchemaBuilder.create(...).measures([...]).dimensions([...]).dateField(...).build()`.

## Consequences

### Positive
- Onboarding de fontes em minutos vs horas
- Reduz erros de mapeamento manual
- Auto-documentação de campos

### Negative
- Heurísticas podem inferir errado (mitigado pelo painel de confirmação)
- Código gerado é ponto de partida, não final
- Complexidade adicional no framework

## Alternatives Considered

| Alternativa | Por quê rejeitada |
|-------------|-------------------|
| Configuração manual (status quo) | Repetitivo, sujeito a erros |
| Schema do banco (Supabase introspection) | Acopla ao Supabase, não funciona com BIMachine |

## Status note
Aguardando implementação quando houver nova fonte de dados a integrar.
