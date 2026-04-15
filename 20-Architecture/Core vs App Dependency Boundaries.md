---
type: concept
status: stable
project: [capra-ui, capra-analytics]
tags: [architecture, boundaries, framework, dependency-rule]
created: 2026-02-13
related: "[[ADR-UI-015 Dependency Boundaries]], [[Framework-First Visual Ownership]], [[Component Categories (UI vs Containers vs Analytics)]], [[Crate Boundaries]]"
---

# Core vs App Dependency Boundaries

> Regra unidirecional de dependência entre framework (`@capra-ui/core`) e app (`capra-analytics`), reforçada por critério explícito de "onde colocar código novo".

## O que é
Lei de boundary entre framework e app, com cinco regras concretas e um teste decisório (Teste do Segundo Consumidor).

## Por que existe
Auditoria de acoplamento (Sessions 70-80) revelou 35+ violações: código de domínio vazando para o core, lógica duplicada, dependências invertidas. Sem regras documentadas, code review não detecta tudo, e o acoplamento volta. Regra clara → previne reintrodução.

## Como funciona

### Direção
```
app → core   ✅ permitido
core → app   ❌ proibido (zero referências)
```

### O que pertence ao core
Genérico que **qualquer app analítica usaria**: componentes (DataTable, KpiCard, FilterBar, KpiContainer), composables reutilizáveis (useDataQuery, useMeasureEngine, useDataLoader), services (ActionBus, FilterManager, QueryManager), utils (debounce, deepClone, formatValue), tipos (DataAdapter, SchemaConfig), error handling (CapraQueryError, executeAllSettled).

### O que pertence ao app
Específico de domínio ou integração BIMachine: schemas de negócio (DATA_SOURCE_GOLD, FILTER_IDS), constantes, composables de feature (useLojas, useVendedores), utils de domínio (normalizePayload, extractCellValue), integração com host (getBIMachineFilters), cálculos de negócio, pages.

### Teste do Segundo Consumidor
> "Se outra app usaria, vai pro core. Se não, fica na app."

Antes de mover qualquer código: uma app de RH analytics usaria isso? Logística? Sim → core. Não → app.

### Hierarquia interna do app
```
pages → composables → schemas → core
```
Nunca inverter (schema importando de composable, composable de page, core de qualquer camada).

## Relações
- `[[ADR-UI-015 Dependency Boundaries]]` — decisão
- `[[Framework-First Visual Ownership]]` — mesma filosofia no eixo visual
- `[[Component Categories (UI vs Containers vs Analytics)]]` — boundaries internos do core
- `[[Crate Boundaries]]` — equivalente no Rust (bode-core sem I/O, bode-server sem domínio)

## Aplicabilidade externa
Pattern reutilizável em qualquer separação framework/app. Permite publicar core como pacote npm independente.
