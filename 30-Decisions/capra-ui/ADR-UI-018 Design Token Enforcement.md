---
type: decision
status: stable
project: [capra-ui, capra-analytics]
tags: [adr, capra-ui, tokens, theming, dark-mode]
created: 2026-02-24
related: "[[ADR-UI-010 Theme System]], [[ADR-UI-019 Framework First Visual Ownership]], [[Framework-First Visual Ownership]]"
---

# ADR-UI-018 Design Token Enforcement

**Status:** Accepted
**Date:** 2026-02-24

## Context
Auditoria do `DashboardPage.vue` revelou múltiplas violações: hex hardcoded (`#10b981`, `#ef4444`, `#e5a22f`...) em inline styles, `style="background: #fef3c7"` em template para legendas de heatmap, hex literals em formatters de chart. Resultado: dark mode quebra (cores fixas não respondem a `data-theme`); identidade visual exige busca em N arquivos.

## Decision
**Zero hex hardcoded em templates Vue. Zero inline styles em pages.**

| Contexto | Proibido | Obrigatório |
|----------|----------|-------------|
| Template Vue (cor) | `style="color: #ef4444"` | classe CSS com `var(--color-*)` |
| Prop accent-color | `accent-color="#9b2c2c"` | `KPI_COLORS.discount` |
| Formatter de chart (HTML string) | `"color: #10b981"` | `` `color: ${CHART_COLORS.positive}` `` |
| Nova cor KPI | `const C = '#...'` local | adicionar em `tokens.css` + `KPI_COLORS` |

### Estrutura
- `capra-ui/src/styles/tokens.css` — fonte canônica (`--color-kpi-*`, `--color-heatmap-*`, `--color-trend-*`, `--color-brand-*`)
- `capra-analytics/src/constants.ts`:
  - `KPI_COLORS` — mapeia para `var(--color-kpi-*)` em props de KpiCard
  - `CHART_COLORS` — hex mirror dos tokens APENAS para formatters de chart

### Exceções (e únicas)
`CHART_COLORS` e `KPI_CHART_PRESETS` — bibliotecas de chart (ECharts/Chart.js) não interpolam CSS vars em strings HTML. Hex nunca aparece fora desses dois objetos.

## Consequences

### Positive
- Dark mode funciona automaticamente em toda UI
- Mudança de identidade visual = alterar `tokens.css`
- Code review trivial: violation = hex literal no template

### Negative
- `CHART_COLORS` é mirror manual — pode desincronizar (atualizar tokens E `CHART_COLORS` juntos)
- Requer disciplina no review (checklist AP-15)

## Alternatives Considered

| Alternativa | Por quê rejeitada |
|-------------|-------------------|
| CSS-in-JS / styled components | Já usa Tailwind v4 + CSS vars; over-engineering |
| PostCSS plugin para extrair vars | Complexity desnecessária no tamanho atual |
