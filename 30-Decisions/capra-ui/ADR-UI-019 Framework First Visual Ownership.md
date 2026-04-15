---
type: decision
status: stable
project: [capra-ui, capra-analytics]
tags: [adr, capra-ui, framework, ownership, design-system]
created: 2026-02-24
related: "[[Framework-First Visual Ownership]], [[ADR-UI-010 Theme System]], [[ADR-UI-015 Dependency Boundaries]], [[ADR-UI-018 Design Token Enforcement]]"
---

# ADR-UI-019 Design System Contract — Framework-First Visual Ownership

**Status:** Accepted
**Date:** 2026-02-24

## Context
capra-ui é framework de componentes para dashboards analíticos. Filosofia análoga ao Bootstrap: **framework define toda a estrutura visual** (espaçamento, tipografia, bordas, cores via tokens, variantes). App consumidora **nunca reimplementa nem sobrescreve** o visual dos componentes do framework.

Caso sintomático: `BaseButton` com estilos em classes Tailwind não processadas pelo content scan do Tailwind v4 em `node_modules` → botões sem cor, sem border-radius, sem padding. Tentativa de "consertar" via `:deep(button)` no app **agravou** a violação ao invés de corrigir a raiz.

## Decision

### Lei de Ownership Visual
- **capra-ui (framework):** estrutura, espaçamento, tipografia, variantes; tokens em `tokens.css`; estilos 100% em `<style scoped>` com `var(--*)`.
- **capra-analytics (app):** `theme.css` SOBRESCREVE tokens de cor (`--color-brand-*`); seleciona variantes via props; injeta CONTEÚDO via slots; CSS próprio APENAS para layout de página e conteúdo de slots.

### App PODE
- Mudar paleta (`theme.css` overrides)
- Escolher variante via prop (`variant="accent"`)
- Injetar conteúdo em slots
- Layout próprio scoped na page (`.loja-detail__grid`)

### App NÃO PODE
- `:deep(button) { ... }` — corrigir no framework
- `.btn-override { background: red }` — criar variant ou usar accent
- `<Button style="border-radius: 0">` — criar variant `square` no framework
- Qualquer scoped CSS sobrescrevendo componente do framework

### Implementação no framework
Componente que precisa de cor DEVE usar `<style scoped>` com CSS custom properties — NUNCA classes Tailwind para cores (Tailwind v4 não scaneia `node_modules`):

```vue
<!-- ✅ CORRETO -->
<style scoped>
.base-btn--accent {
  background: var(--color-brand-highlight);
}
</style>
```

### Customização na app
`capra-analytics/src/theme.css` sobrescreve tokens; todos os componentes respondem automaticamente.

## Consequences

### Positive
- Consistência visual garantida
- Impossível "overrides locais que crescem sem controle"
- Theming via 1 arquivo CSS muda toda a paleta
- Manutenção clara: problema visual → corrigir no framework
- Dark mode via override em `[data-theme="dark"]` no theme.css

### Negative
- Variantes novas precisam ir ao framework (não ad-hoc no app)
- PR ao capra-ui necessário para qualquer mudança visual de componente

## Related
- `[[ADR-UI-010 Theme System]]` — mecanismo `theme.css` + `[data-theme]`
- `[[ADR-UI-015 Dependency Boundaries]]` — direção de imports
- AP-14 (Framework-First — reimplementação completa)
- AP-15 (Design Token Enforcement — `[[ADR-UI-018 Design Token Enforcement]]`)
- AP-16 (CSS Override de componentes do framework)
