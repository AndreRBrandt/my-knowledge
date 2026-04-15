---
type: concept
status: stable
project: [capra-ui]
tags: [architecture, capra-ui, theme, dark-mode, css]
created: 2026-02-10
related: "[[ADR-UI-010 Theme System]], [[Framework-First Visual Ownership]]"
---

# Theme System

> Sistema de tema baseado em `[data-theme]` attribute no `<html>` + tokens CSS sobreescrevíveis, com suporte a `light`/`dark`/`system`.

## O que é
- **`useTheme`** composable que aplica `document.documentElement.dataset.theme` com modo resolvido
- **`tokens.css`** define paleta light (default)
- **`dark.css`** opt-in via import, override via `[data-theme="dark"] { --color-text: ...; }`
- **Brand colors** (`--color-brand-*`) NÃO são sobrescritas em dark — identidade preservada

## Por que existe
Framework precisa de dark mode sem acoplar componentes a um modo específico, sem quebrar identidade visual, e funcionando em CSS puro (sem depender de JS após primeiro render).

## Como funciona

### Atributo no `<html>`
```css
[data-theme="dark"] {
  --color-text: #e5e7eb;
  --capra-text: #e5e7eb;
  /* ... */
}
```

### Composable
- Tenta `inject` do plugin primeiro; fallback cria instância local (singleton via injection key)
- Modo `system` escuta `matchMedia('(prefers-color-scheme: dark)')` em tempo real
- Persistência: `useConfigState` (debounce + localStorage)

### Por que `[data-theme]` (não classe `.dark`)
- Não conflita com Tailwind `dark:` nem utility classes
- Semântico, funciona com CSS-only selectors
- Evita conflitos com outras libs

### Dois sistemas de tokens sincronizados
- `--color-*` — design tokens base (`tokens.css`)
- `--capra-*` — tokens de componente (`theme.css`)
- Dark mode override ambos

## Relações
- `[[ADR-UI-010 Theme System]]` — decisão
- `[[Framework-First Visual Ownership]]` — apps customizam pelo `theme.css` (override de tokens), não scoped CSS
- `[[ADR-UI-018 Design Token Enforcement]]` — sem hex hardcoded = dark funciona automaticamente

## Limitações conhecidas
- `dark.css` duplica todos tokens — manutenção manual ao adicionar novos
- Componentes com fallback hex hardcoded (em `var(--x, #default)`) podem não respeitar dark se token não estiver definido
