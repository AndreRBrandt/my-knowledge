---
type: decision
status: stable
project: [bode-api]
tags: [adr, project-management, github, workflow]
created: 2026-04-08
related: "[[bode-api]]"
---

# ADR-BODE-008 GitHub Projects para Gestão de Projeto

**Status:** Accepted
**Date:** 2026-04-08

## Context
Precisávamos de uma ferramenta para gestão de backlog técnico (specs, epics, stories, tasks) com API access para automação via Claude Code CLI.

Opções avaliadas:
- **ClickUp (workspace corporativo)** — Sem permissão de API (guest). Sem autonomia.
- **ClickUp (workspace pessoal Free)** — 1 space, sem Gantt, sem automações.
- **Linear Free** — Bom, mas mais uma ferramenta externa.
- **Plane (self-hosted)** — Mais um container na VPS.
- **GitHub Projects** — Gratuito, ilimitado, integrado ao código.

## Decision
Usar **GitHub Projects** na org `bodedono`.

**Motivos:**
1. **Zero custo** — ilimitado no plano Free do GitHub
2. **Integração nativa** — PRs linkam automaticamente a tasks, merge fecha issue
3. **API via gh CLI** — Claude Code cria/atualiza tasks direto do terminal
4. **Sem ferramenta extra** — já usamos GitHub para código e CI/CD
5. **Autonomia total** — André é admin da org bodedono

**Estrutura:**
```
Project: Bode Analytics Platform (org: bodedono)
├── Custom Fields: Status, Priority, Project, Kind, Phase
├── Views: Board (kanban), Table, Roadmap
└── Epics como draft issues com DoD checklists
```

**Workflow:**
1. André define requisitos
2. Claude redige spec
3. André valida/ajusta
4. Claude cria backlog no GitHub Projects (Epic → Story → Task)
5. André implementa (Claude = tutor)
6. Claude atualiza status
7. Documentar na wiki + roadmap

## Consequences
- Sem Gantt nativo (usar Roadmap view como alternativa)
- Draft issues não linkam a repos até serem convertidas em issues reais
- Sprints via iteration fields (configurar manualmente no browser)

## Alternatives Considered

| Alternative | Why rejected |
|-------------|--------------|
| ClickUp (workspace corporativo) | Sem permissão de API (guest). Sem autonomia. |
| ClickUp (workspace pessoal Free) | 1 space, sem Gantt, sem automações. |
| Linear Free | Bom, mas mais uma ferramenta externa. |
| Plane (self-hosted) | Mais um container na VPS. |

## References
- GitHub Project: `https://github.com/orgs/bodedono/projects/1`
