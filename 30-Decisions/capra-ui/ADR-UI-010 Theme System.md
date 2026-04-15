---
type: decision
status: stable
project: [capra-ui]
tags: [adr, capra-ui, theme, dark-mode]
created: 2026-02-10
related: "[[Theme System]], [[ADR-UI-018 Design Token Enforcement]], [[ADR-UI-019 Framework First Visual Ownership]]"
---

# ADR-UI-010 Theme System (Dark Mode)

**Status:** Accepted
**Date:** 2026-02-10

## Context
Framework precisa de dark mode. Existem dois conjuntos de variáveis CSS a sincronizar: `tokens.css` (`--color-*`, design tokens base) e `theme.css` (`--capra-*`, tokens de componente). Dark mode deve sobrescrever ambos sem afetar brand colors.

## Decision
**`[data-theme]` attribute no `<html>`** — `useTheme` aplica `document.documentElement.dataset.theme` com modo resolvido (`light`/`dark`/`system`). CSS de dark mode usa seletor `[data-theme="dark"]` para override.

Decisões específicas:
- `[data-theme]` sobre classe CSS — semântico, não conflita com utility classes
- Brand colors (`--color-brand-*`) NÃO sobrescritas em dark — identidade preservada
- `dark.css` separado, opt-in via import
- Singleton via injection key (fallback cria local)
- Modo `system` escuta `matchMedia('(prefers-color-scheme: dark)')` em tempo real
- Persistência via `useConfigState` (debounce + localStorage)

## Consequences

### Positive
- CSS-only approach (funciona sem JS se pré-setado)
- Dois sistemas de tokens sincronizados
- Brand inalterado
- Reage a mudança do OS em tempo real

### Negative
- `dark.css` duplica todos os tokens — manutenção manual ao adicionar
- `[data-theme]` pode conflitar com outras libs que usem mesma abordagem
- Componentes com fallback hex hardcoded podem não respeitar dark

## Alternatives Considered

| Alternativa | Por quê rejeitada |
|-------------|-------------------|
| Classe CSS `.dark` | Conflita com Tailwind `dark:` e utilities |
| `prefers-color-scheme` only | Não permite toggle manual |
| `color-scheme` CSS prop | Suporte limitado p/ override granular |
| Variáveis HSL dinâmicas | Complexidade desnecessária |

## References
- `[[Theme System]]` — concept
