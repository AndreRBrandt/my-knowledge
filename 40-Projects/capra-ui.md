---
type: hub
status: living
project: capra-ui
tags: [project, hub, capra-ui, frontend, framework]
created: 2026-04-15
---

# capra-ui

> Framework Vue 3 de componentes para dashboards analíticos. Pacote `@capra-ui/core` (público em `github.com/AndreRBrandt/capra-ui`).

## Repo
- GitHub: `AndreRBrandt/capra-ui` (público)
- Workspace local: `bi_projects/capra-workspace/capra-ui/`

## Stack
- Vue 3.5+ (Composition API), TypeScript 5.9+ strict, Vite 7.3+, Tailwind v4.1+
- Vitest 4 + Vue Test Utils + Playwright

## Decisions
- `[[ADR-UI-001 Adapter Pattern]]` ... `[[ADR-UI-019 Framework First Visual Ownership]]` (ver `[[Decisions MOC]]`)
- Cross-cutting: `[[GADR-005 Framework-First UI Development]]`

## Architecture concepts
- `[[Adapter Pattern (UI Layer)]]`
- `[[Action Bus]]`
- `[[Schema Builder]]`
- `[[Measure Engine]]`
- `[[Filter Registry]]`
- `[[Theme System]]`
- `[[Domain Containers]]`
- `[[Dimension Discovery]]`
- `[[Component Categories (UI vs Containers vs Analytics)]]`
- `[[Interaction System]]`
- `[[Generic Composables Layer]]`
- `[[Data Loading Patterns]]`
- `[[Framework-First Visual Ownership]]`
- `[[Core vs App Dependency Boundaries]]`

## Domain (negócio que o framework expõe genericamente)
Não tem domain próprio — framework é agnóstico. Domain do consumidor (capra-analytics):
- `[[Filial]]`, `[[Empresa]]`
- `[[Roles]]`, `[[Permissions]]`, `[[RBAC Cascade Model]]`

## Composição de camadas

```
Page (capra-analytics)
  └── Domain Container (capra-ui)        ← KpiContainer, DataTableContainer
        └── Primitives (capra-ui)         ← AnalyticContainer, KpiCard, KpiGrid, etc.
              └── Generic Composables     ← useAnalyticData, useDrillStack, ...
                    └── Services          ← ActionBus, FilterManager, QueryManager
                          └── Adapter     ← BIMachineAdapter / WorkersAdapter / MockAdapter
```

## Comandos principais
```bash
# do workspace root:
pnpm test:core                            # testes do framework apenas
pnpm --filter @capra-ui/core build        # build da lib

# dentro de capra-ui/:
pnpm test                                 # vitest
```

## Estado atual
- 19 ADRs ativas (algumas proposed: ADR-UI-008 Filter Registry, ADR-UI-017 Endpoint Schema Discovery)
- Componentes nas 4 categorias (UI, Containers, Analytics, Layout)
- Services (ActionBus, FilterManager, QueryManager) implementados; FilterManager não-usado pelo app ainda (ADR-UI-009 deferiu)
- Theme system com dark mode (`[data-theme]`)
- Migração contínua para framework-first (eliminação progressiva de violações AP-14/15/16)

## Roadmap
- 📋 `[[capra-ui Improvements Plan]]` (PLACEHOLDER) — workstream futuro de melhoria + documentação do framework. Aciona quando bode-api Phase 2 estiver em Sprint 3+ ou houver demanda de produto.
