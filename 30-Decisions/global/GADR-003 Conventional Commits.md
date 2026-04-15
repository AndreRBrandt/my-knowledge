---
type: decision
status: stable
scope: workspace
tags: [decision, global, workspace, git, commits]
created: 2026-02-15
related: "[[GADR-002 Trunk-Based Development]]"
---

# GADR-003 Conventional Commits with Project Scopes

**Status:** Accepted
**Date:** 2026-02-15

## Context
Histórico precisa ser legível pra retomada de contexto entre sessões. Sem padrão, commits viram `update`, `fix bug`, `WIP`. Difícil entender mudanças por scope ou tipo.

## Decision
Padrão `type(scope): description`. Scope obrigatório.

**Tipos:** `feat`, `fix`, `refactor`, `perf`, `test`, `docs`, `chore`, `ci`

**Scopes por projeto:**

| Projeto | Scopes |
|---------|--------|
| bode-api | `core`, `db`, `server`, `sync`, `ci`, `deps`, `config` |
| capra-ui | `ui`, `analytics`, `containers`, `filters`, `composables`, `adapters`, `services` |
| capra-analytics | `dashboard`, `domain`, `data`, `integration`, `app` |
| bode-analytics-api | `routes`, `services`, `pipeline`, `middleware`, `config` |

Validação via `commitlint` + `husky` pre-commit hook.

## Consequences

### Positive
- `git log` filtra por tipo (`--grep "^feat"`) ou scope
- Squash merge gera mensagem coerente (PR title = squash message)
- Changelog automatizável a partir do log
- Onboarding óbvio (novo dev vê o padrão e replica)

### Negative
- Scope errado vira ruído (mitigado por commitlint config por projeto)
- Hook pode atrasar commit em segundos

## References
- `[[GADR-002 Trunk-Based Development]]`
