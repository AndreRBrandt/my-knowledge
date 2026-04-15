---
type: decision
status: stable
project: [capra-ui]
tags: [adr, capra-ui, layout, css-grid]
created: 2026-02-18
related: "[[ADR-UI-011 Domain Containers]], [[ADR-UI-018 Design Token Enforcement]]"
---

# ADR-UI-016 KPI Grid Layout & Container Theming

**Status:** Accepted
**Date:** 2026-02-18

## Context
KPI cards precisavam: altura fixa uniforme + largura uniforme + adaptação ao container + min/max width configuráveis. AnalyticContainer também precisava de header destacado (highlight) sem acoplar cores específicas.

## Decision

### KpiGrid — CSS Grid `auto-fit` + `minmax(min, 1fr)` + `max-width` nos filhos
- `auto-fit` colapsa tracks vazias → cards crescem
- `minmax(min, 1fr)` → mínimo garantido, expansão até 1fr
- `max-width` no child → limita esticamento em linhas incompletas
- `grid-auto-rows` → altura fixa

Props (CSS vars opcionais): `--kpi-gap`, `--kpi-min-width` (200px), `--kpi-max-width`, `--kpi-card-height` (110px).

### AnalyticContainer — `highlightHeader` prop + cascade de tokens
```css
background-color: var(--analytic-header-bg, var(--color-brand-secondary, #3a1906));
```
Camadas: override por instância → token global → fallback hex (nunca usado se tokens carregados).

### Padrão geral de cores
Brand tokens (`var(--color-brand-*, fallback)`) em TODO estado interativo. Hex direto sem token = ❌.

### KpiData.history como tipo genérico
`Array<{ label: string; value: number }>` — core não sabe origem (MDX/REST/mock); composable `useKpiTrend` na app busca e transforma.

## Consequences

### Positive
- Zero acoplamento core/app — tudo configurável
- Theming via CSS vars (consumer muda `tokens.css`)
- Prop-driven, slots para extensão (`#detail-modal`, `#card`, `#actions`)

### Negative
- `auto-fit` em linhas incompletas pode variar largura (mitigado por `maxCardWidth`)
- Removida prop `columns` (usar `minCardWidth` indiretamente)

## Alternatives Considered

| Alternativa | Por quê rejeitada |
|-------------|-------------------|
| `auto-fill` | Cards não esticam quando há poucos |
| Flexbox `flex: 1 0 min-width` | Última linha incompleta varia largura |
| `repeat(N, 1fr)` fixo | Não responsivo, precisa JS |
| `:deep` na page consumidora | Acopla page ao DOM interno |
