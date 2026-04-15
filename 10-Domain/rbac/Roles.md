---
type: concept
status: stable
domain: rbac
project: [bode-api]
tags: [domain, rbac, auth, permissions]
created: 2026-04-14
related: "[[Permissions]], [[RBAC Cascade Model]], [[User Permissions Override]], [[Filial]]"
---

# Roles

> Papel do usuário no sistema. Define o conjunto **default** de permissões (que pode ser sobrescrito a nível de usuário, sempre para mais restritivo).

## O que é
Tabela `roles` na OLTP do `bode-api`:

```sql
CREATE TABLE roles (
    id   SERIAL PRIMARY KEY,
    slug TEXT NOT NULL UNIQUE,
    name TEXT NOT NULL
);
```

`slug` (estável, usado em código) e `name` (display, editável) separados intencionalmente — renomear não quebra referências.

## Roles atuais (seed)

| id | slug | name | Visão de filiais | Defaults de permissão |
|----|------|------|------------------|----------------------|
| 1 | `adm` | Administrador | Todas (sem `user_filiais`) | absolute em revenue/cost/margin, export allowed |
| 2 | `diretor` | Diretor | Todas (sem `user_filiais`) | absolute em revenue/cost/margin, export allowed |
| 3 | `gestor_filial` | Gestor de Filial | Subset via `user_filiais` | percentage em revenue/margin, hidden em cost, export allowed |

## Por que role é configurável (não enum)
- Decisão arquitetural: roles mudam com frequência. Admin panel cria/edita roles sem deploy.
- Implementação no Rust: `role: String` (não enum) — total flexibilidade.

## Cascata
1. `roles` define defaults via `role_permissions`
2. `[[User Permissions Override]]` pode SOBRESCREVER apenas para mais restritivo (validado no write, não no read)

## Visão de filiais (regra explícita)
- `adm`/`diretor` SEM linha em `user_filiais` → vê TODAS as filiais
- `gestor_filial` DEVE ter linhas em `user_filiais` → vê apenas subset

Esta regra é semântica: ausência de filiais = "todas" para roles administrativos; subset obrigatório para roles operacionais.

## Onde é usado
- `[[bode-api]]` — JWT carrega `role`, AuthUser expõe `role: String`
- Frontend filtra UI baseado em `permissions` resolvidas (não no role direto)
