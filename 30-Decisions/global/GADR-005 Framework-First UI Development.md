---
type: decision
status: stable
scope: workspace
tags: [decision, global, workspace, ui, framework]
created: 2026-02-24
related: "[[Framework-First Visual Ownership]], [[ADR-UI-019 Framework First Visual Ownership]], [[ADR-UI-015 Dependency Boundaries]], [[Core vs App Dependency Boundaries]]"
---

# GADR-005 Framework-First UI Development

**Status:** Accepted
**Date:** 2026-02-24

## Context
capra-ui é framework de componentes para dashboards analíticos. capra-analytics consome o framework. Tendência natural: quando UI tem fricção, a app "conserta localmente" via `:deep()`, classes override, scoped CSS sobrescrevendo componente do framework. Isso cria divergências invisíveis e bloqueia theming centralizado.

## Decision
**Lei workspace-wide: o framework é dono do visual dos componentes; a app só customiza via mecanismos previstos.**

App PODE: trocar paleta via `theme.css`, escolher variante via prop, injetar conteúdo via slot, ter CSS próprio para layout de página.

App NÃO PODE: `:deep(button)`, classes override de componente do framework, `style="..."` inline em componente do framework, reimplementar componente que existe no framework.

Quando app precisa de visual diferente: PR para o framework (nova variant, nova prop, fix do bug). Nunca monkey-patch local.

Aplica-se a TODA UI cross-project: capra-analytics hoje, qualquer app futura amanhã.

## Consequences

### Positive
- Consistência visual garantida cross-project
- Theming via 1 arquivo CSS muda toda a paleta
- Manutenção clara: problema visual → corrigir no framework
- Onboarding: novo dev tem regra explícita

### Negative
- Variante nova exige ciclo PR no framework (não ad-hoc)
- Disciplina em code review obrigatória

## References
- `[[Framework-First Visual Ownership]]` — concept
- `[[ADR-UI-019 Framework First Visual Ownership]]` — decisão no escopo capra-ui
- `[[ADR-UI-018 Design Token Enforcement]]` — pré-requisito (zero hex hardcoded)
- AP-14, AP-15, AP-16 (anti-patterns documentados)
