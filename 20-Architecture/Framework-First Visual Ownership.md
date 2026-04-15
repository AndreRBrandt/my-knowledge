---
type: concept
status: stable
project: [capra-ui, capra-analytics]
tags: [architecture, capra-ui, framework, design-system, ownership]
created: 2026-02-24
related: "[[ADR-UI-019 Framework First Visual Ownership]], [[Theme System]], [[Core vs App Dependency Boundaries]], [[ADR-UI-018 Design Token Enforcement]]"
---

# Framework-First Visual Ownership

> Lei arquitetural: o framework de componentes é dono do visual dos seus componentes; a app só customiza via mecanismos previstos (tokens, props, slots).

## O que é
Filosofia análoga ao Bootstrap: **capra-ui define toda a estrutura visual** (espaçamento, tipografia, bordas, cores via tokens, variantes); **capra-analytics nunca reimplementa nem sobrescreve** o visual.

## Por que existe
Sem essa lei, "CSS local creeping" cresce nas pages: `:deep(button)`, classes override (`.btn-fix`), estilos inline. Resultado: divergências invisíveis entre pages, impossível manter consistência, identidade visual exige busca em N arquivos.

Caso sintomático: BaseButton com classes Tailwind no framework não funcionavam (Tailwind v4 não scaneia `node_modules`). Tentativa de "corrigir" via `:deep(button)` na app **agravou** ao invés de corrigir a raiz no framework.

## Como funciona

### App PODE
| Ação | Mecanismo |
|------|-----------|
| Mudar paleta | `theme.css` override de tokens (`--color-brand-*`) |
| Variante visual | Prop do componente (`variant="accent"`) |
| Conteúdo | Slot (`<template #header>...</template>`) |
| Layout próprio | Scoped CSS na page, ou em conteúdo de slots |

### App NÃO PODE
| Anti-pattern | Correto |
|--------------|---------|
| `:deep(button) { gap: ... }` | Corrigir gap no BaseButton do framework |
| `.btn-override { background: red }` | Criar variant ou usar accent |
| `<Button style="border-radius: 0">` | Criar variant `square` no framework |
| Scoped CSS sobrescrevendo componente do framework | PR ao framework ou usar variant/slot |

### Implementação no framework
Componentes que precisam de cor DEVEM usar `<style scoped>` com `var(--*)`. **NUNCA classes Tailwind para cores** (Tailwind v4 não processa `node_modules`):

```vue
<style scoped>
.base-btn--accent {
  background: var(--color-brand-highlight);
}
</style>
```

## Relações
- `[[ADR-UI-019 Framework First Visual Ownership]]` — decisão
- `[[Theme System]]` — mecanismo `theme.css` + `[data-theme]`
- `[[Core vs App Dependency Boundaries]]` — direção de imports complementa
- `[[ADR-UI-018 Design Token Enforcement]]` — sem hex hardcoded é pré-requisito

## Aplicabilidade externa
Princípio reutilizável em qualquer biblioteca de componentes que tenha consumidores que tenderiam a "consertar localmente". A lei previne entropy do design system.
