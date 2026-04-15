---
type: concept
status: stable
domain: rbac
project: [bode-api]
tags: [domain, rbac, override, cascade]
created: 2026-04-14
related: "[[Roles]], [[Permissions]], [[RBAC Cascade Model]]"
---

# User Permissions Override

> Mecanismo que permite sobrescrever a permissão default da role para um usuário específico — sempre tornando-a mais restritiva, nunca mais permissiva.

## O que é
Tabela:

```sql
CREATE TABLE user_permissions (
    user_id             UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    permission_value_id INT NOT NULL REFERENCES permission_values(id) ON DELETE CASCADE,
    PRIMARY KEY (user_id, permission_value_id)
);
```

Usuário pode ter um `permission_value_id` que SUBSTITUI o default da sua role.

## Regra: cascata restritiva (write-time validation)
**Override só pode ser MAIS RESTRITIVO que o default da role.** Validado no momento da escrita (admin panel), não na leitura.

```
default da role (level=1) → user override (level=2) ✅ válido (mais restritivo)
default da role (level=2) → user override (level=1) ❌ rejeitado (menos restritivo)
```

Por que validar no write: leitura roda em todo request; manter rápido (`COALESCE` simples). A admin panel é o gate.

## Resolução em runtime (read-time)
```sql
-- Pseudo: para cada permission, prefere user override sobre role default
SELECT p.slug AS permission, COALESCE(uv.slug, rv.slug) AS value
FROM permissions p
LEFT JOIN role_permissions rp ON ...
LEFT JOIN permission_values rv ON rp.permission_value_id = rv.id
LEFT JOIN user_permissions up ON up.user_id = $1 AND ...
LEFT JOIN permission_values uv ON up.permission_value_id = uv.id
WHERE ...
```

(Implementação real em `crates/bode-db/src/users.rs`.)

## Casos de uso
- Diretor (`view_revenue=absolute`) que não deve ver custos: override `view_cost=hidden`
- Gestor de filial Y que NÃO pode exportar: override `export_data=denied`

## Onde é usado
- `[[bode-api]]` — query `getUserPermissions` resolve cascade
- Admin panel (futuro) gerencia overrides com validação de level
