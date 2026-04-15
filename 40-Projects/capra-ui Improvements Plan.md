---
type: plan
status: placeholder
project: [capra-ui]
tags: [frontend, framework, planning, capra-ui]
created: 2026-04-15
related: "[[capra-ui]], [[bode-api Phase 2 Deploy Plan]]"
---

# capra-ui — Improvements Plan

> **Status:** PLACEHOLDER — não iniciado.
>
> Workstream paralelo ao backend (`[[bode-api Phase 2 Deploy Plan]]`). Foi reservado neste arquivo para que o trabalho exista no índice e possa ser acionado quando priorizado.

## Why this exists (placeholder rationale)

Conforme conversa de 2026-04-15 (S237 follow-up), André sinalizou que além da migração TS→Rust + Supabase→self-hosted, há um **terceiro eixo** de trabalho previsto: melhoria e documentação do frontend + aprimoramento do framework `capra-ui`.

Este placeholder reserva o espaço no vault para que:
1. Decisões e ideias relacionadas tenham um destino canônico (não se percam em conversas)
2. Quando o workstream começar de fato, exista um ponto de partida com o contexto já carregado
3. O hub `[[capra-ui]]` possa apontar pra cá

## Tópicos prováveis (a confirmar quando ativarmos)

Lista não-comprometida — coisas que **podem** entrar quando expandirmos:

### Documentação
- Storybook ou docs site dedicado (Histoire? VitePress?) pros componentes do framework
- Inventário completo dos componentes (UI / Containers / Analytics / Layout) com props, slots, eventos, exemplos
- Guia de "como criar um novo componente" alinhado a Framework-First (AP-14)
- Migration guide quando houver breaking changes
- Cookbook de padrões comuns no app consumidor (capra-analytics)

### Aprimoramento do framework
- Resolver ADRs proposed pendentes (ex.: ADR-UI-008 Filter Registry, ADR-UI-017 Endpoint Schema Discovery)
- Auditoria de acessibilidade (WCAG AA) nos componentes core
- Performance budgets + tree-shaking dos bundles
- Test coverage targets por categoria (UI primitives ≥ 95%, Containers ≥ 80%)
- Theme tokens system (cores, spacing, typography) padronizado e documentado
- Type-safety end-to-end (props ↔ adapters ↔ services)

### Tooling / DX
- Plugin Vite custom pra catch violações de Framework-First em dev
- ESLint rules customizadas pros invariants do framework
- Pre-commit hook validando classes Tailwind contra design tokens

### Quality gates do app consumidor
- Quando houver design tokens canônicos, varrer `capra-analytics` por classes Tailwind hardcoded e migrar
- Substituir progressivamente componentes app-only por equivalentes do framework

## Out of scope (sempre)
- RBAC implementation no frontend (esperar backend Epic #8)
- Migração de adapters legados (BIMachine descontinuado, ver `[[GADR-008 BIMachine Deprecation]]`)

## Quando ativar este plano
- Quando `bode-api` Phase 2 estiver em Sprint 3+ (deploy estável) → desbloqueia bandwidth
- Ou quando aparecer demanda específica de produto que dependa de capra-ui (ex.: Capra Checklist precisar de novos componentes)

## Próximos passos quando ativar
1. Substituir status `placeholder` → `draft`
2. Decidir os 3-5 tópicos prioritários da lista acima
3. Estruturar como Epic → Story → Task → DoD (mesmo modelo do plano de bode-api)
4. Validar com André
5. Criar issues no repo `AndreRBrandt/capra-ui`
