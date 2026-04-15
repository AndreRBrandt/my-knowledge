---
type: decision
status: stable
scope: workspace
tags: [decision, global, workspace, git, branching]
created: 2026-02-15
related: "[[GADR-003 Conventional Commits]]"
---

# GADR-002 Trunk-Based Development with Squash Merge

**Status:** Accepted
**Date:** 2026-02-15

## Context
Equipe pequena (1 dev + LLM). Branching elaborado (gitflow, dev/staging/main) introduz overhead sem benefício correspondente. Mas push direto em `main` quebra rastreabilidade e elimina chance de validação automática.

## Decision
Trunk-based com PR + squash merge:

1. `git checkout -b feature/<name>` from `main`
2. Commits atômicos (`type(scope): description`)
3. PR para `bodedono/*` repos → squash merge
4. Merge em `main` → auto-deploy homolog
5. Tag `v1.x.x` em main → auto-deploy prod

**Proibido:**
- Push direto em `main`
- `--no-verify` (skip hooks)
- Merge sem CI verde
- Branches `dev`/`staging` paralelos

## Consequences

### Positive
- Histórico linear e legível (1 squash = 1 feature)
- CI valida toda mudança antes de chegar a main
- Auto-delete de branch após merge (sem clutter)
- Branch protection em `main` aplica regras

### Negative
- Rebase ocasional necessário se main avança rapidamente
- Squash perde commits intermediários (vivem no PR para histórico)

## Alternatives Considered

| Alternativa | Por quê rejeitada |
|-------------|-------------------|
| Gitflow (dev + main + release) | Overkill p/ equipe pequena |
| Push direto em main | Sem CI gate, sem rastreabilidade |
| Merge commit (não squash) | Histórico polui com commits WIP |

## References
- `[[GADR-003 Conventional Commits]]` — padrão dos commits
