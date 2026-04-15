---
type: decision
status: stable
project: [capra-ui, capra-analytics]
tags: [adr, boundaries, architecture, framework]
created: 2026-02-13
related: "[[Core vs App Dependency Boundaries]], [[ADR-UI-019 Framework First Visual Ownership]]"
---

# ADR-UI-015 Dependency Boundaries (Core vs App)

**Status:** Accepted
**Date:** 2026-02-13

## Context
Auditoria de acoplamento (Sessions 70-80) identificou 35+ problemas: código de domínio vazando para o core, lógica duplicada em múltiplos composables, dependências fluindo na direção errada. Sem regras documentadas, o acoplamento volta.

## Decision

### Regra 1 — Direção
```
app → core   ✅ permitido
core → app   ❌ proibido
```

### Regra 2 — Pertence ao core (`@capra-ui/core`)
Componentes/composables/services genéricos: DataTable, KpiCard, FilterBar, KpiContainer, useDataQuery, ActionBus, FilterManager, debounce, DataAdapter, CapraQueryError, executeAllSettled. Critério: **resolve problema genérico que qualquer app analítica teria**.

### Regra 3 — Pertence ao app (`capra-analytics`)
Schemas de negócio (DATA_SOURCE_GOLD, FILTER_IDS), constantes de domínio, composables de feature (useLojas, useVendedores), utils de domínio (normalizePayload, extractCellValue), integração com host (getBIMachineFilters), cálculos de negócio, pages. Critério: **específico do domínio ou da integração BIMachine**.

### Regra 4 — Teste do Segundo Consumidor
> "Se outra app usaria, vai pro core. Se não, fica na app."

Pergunta antes de mover: uma app de RH analytics usaria isso? Logística? Sim → core. Não → app.

### Regra 5 — Hierarquia interna do app
```
pages → composables → schemas → core
```
Nunca inverter.

## Consequences

### Positive
- Previne reintrodução de acoplamento
- Onboarding claro
- Core testável isoladamente
- Core publicável como pacote npm independente

### Negative
- Overhead de decisão a cada novo código
- Duplicação temporária possível antes de promover ao core

## References
- `[[Core vs App Dependency Boundaries]]` — concept
- Sessions 70-80 (auditoria de acoplamento)
