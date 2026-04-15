---
type: concept
status: stable
domain: rbac
project: [bode-api]
tags: [domain, rbac, permissions, levels]
created: 2026-04-14
related: "[[Roles]], [[User Permissions Override]], [[RBAC Cascade Model]]"
---

# Permissions

> Catálogo de permissões + valores possíveis com ordenação por nível de restrição. Modelo flexível: novas permissões viram linhas, não código.

## O que é
Duas tabelas:

```sql
CREATE TABLE permissions (
    id          SERIAL PRIMARY KEY,
    slug        TEXT NOT NULL UNIQUE,   -- 'view_revenue', 'export_data'
    name        TEXT NOT NULL,           -- display
    description TEXT
);

CREATE TABLE permission_values (
    id            SERIAL PRIMARY KEY,
    permission_id INT NOT NULL REFERENCES permissions(id) ON DELETE CASCADE,
    slug          TEXT NOT NULL,
    name          TEXT NOT NULL,
    level         INT NOT NULL,          -- maior = mais restritivo
    UNIQUE (permission_id, slug),
    UNIQUE (permission_id, level)
);
```

## Catálogo atual (seed)

| Permission slug | Valores (slug → level) |
|-----------------|------------------------|
| `view_revenue` | absolute=1, percentage=2, hidden=3 |
| `view_cost` | absolute=1, percentage=2, hidden=3 |
| `view_margin` | absolute=1, percentage=2, hidden=3 |
| `export_data` | allowed=1, denied=2 |

## Conceito de `level`
**Maior level = mais restritivo.** Permite cascata determinística: usuário pode SOBRESCREVER seu permission_value para um level **maior ou igual** ao default da role; nunca menor.

Exemplo: `diretor` tem `view_revenue=absolute(1)`. Diretor X pode ter override para `view_revenue=hidden(3)` (mais restritivo, válido). Não pode ter `absolute` se a role for `gestor_filial` (que já vem com `percentage(2)`).

## HashMap em runtime
Após resolução pela cascade, `AuthUser.permissions: HashMap<String, String>`:
```rust
{
  "view_revenue": "absolute",
  "view_cost": "absolute",
  "view_margin": "absolute",
  "export_data": "allowed",
}
```

Frontend lê esse map e oculta/transforma valores na UI.

## Onde é usado
- `[[bode-api]]` — query `getUserPermissions(user_id)` faz COALESCE(user_permissions, role_permissions)
- `[[capra-analytics]]` — UI esconde KPI de custo se `view_cost === 'hidden'`, exibe `%` se `percentage`

## Adicionar nova permissão
1. INSERT em `permissions` (slug + name + description)
2. INSERT em `permission_values` (definir todos os valores possíveis com levels)
3. INSERT em `role_permissions` para defaults por role
4. Frontend lê automaticamente — não precisa deploy de código se já houver UI genérica
