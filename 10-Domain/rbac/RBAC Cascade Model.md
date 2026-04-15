---
type: concept
status: stable
domain: rbac
project: [bode-api]
tags: [domain, rbac, model, mental-model]
created: 2026-04-14
related: "[[Roles]], [[Permissions]], [[User Permissions Override]], [[Filial]], [[Auth Flow JWKS]]"
---

# RBAC Cascade Model

> Modelo mental completo de como permissões e visibilidade de filiais resolvem-se para um usuário, do JWT até o HashMap final entregue ao frontend.

## Visão geral

```
JWT (Supabase Auth, ES256/JWKS)
  └── sub (UUID do user)
        │
        ▼
bode-api (auth middleware)
  ├── valida assinatura (chave pública JWKS, validação offline)
  └── busca user no banco OLTP:
        ├── 1. user (id, email, name, role_id, active)
        ├── 2. filiais visíveis (regra de role abaixo)
        └── 3. permissions (cascade abaixo)
              │
              ▼
        AuthUser { id, email, role: String, filiais: Vec<i32>, permissions: HashMap<String, String> }
              │
              ▼
        injetado em handler como Authenticated(AuthUser)
              │
              ▼
        Frontend usa permissions + filiais para renderizar UI
```

## Resolução de filiais

| Regra | Quem | Resultado |
|-------|------|-----------|
| SEM linha em `user_filiais` E role ∈ {`adm`, `diretor`} | Roles administrativos | TODAS as filiais ativas |
| Linhas em `user_filiais` | Qualquer role | Subset declarado |
| SEM linha + role ≠ administrativa | (estado inválido) | Rejeitar via constraint |

Detalhes em `[[Filial]]` e `[[Roles]]`.

## Resolução de permissions (COALESCE explícito)

Para cada `permission` no catálogo:
1. Existe override em `user_permissions`? → usa o `permission_value` do override
2. Senão, usa `permission_value` do `role_permissions` da role do usuário
3. Senão (sem default na role) → permission ausente do HashMap

Resultado: `HashMap<permission_slug, permission_value_slug>` ex:
```
{
  "view_revenue": "absolute",
  "view_cost": "hidden",       // override do user
  "view_margin": "percentage",
  "export_data": "allowed",
}
```

Detalhes em `[[Permissions]]` e `[[User Permissions Override]]`.

## Validações de invariante
- Override só pode ser **mais restritivo** que o default da role (validado no write)
- `slug` de role/permission/value são UNIQUE — admin panel pode renomear `name` sem quebrar referências
- `level` de permission_values é UNIQUE por permission_id — ordenação determinística

## Frontend
Frontend NUNCA decide permissão; apenas reflete o HashMap recebido:
- `view_cost === 'hidden'` → não renderiza KPI de custo
- `view_revenue === 'percentage'` → exibe % em vez do absoluto
- `export_data === 'denied'` → desabilita botão exportar

Toda decisão de gate é server-side (handler valida `permissions` antes de retornar dados sensíveis).

## Por que separado do OLAP
OLTP (RBAC, auth, configuração) vive em `bode-api` Postgres; OLAP (analítico, dbt) vive em Supabase. Correlação via UUID do user. Razão: workloads diferentes (transacional vs analítico), evolução independente, hardening separado.

## Onde é usado
- `[[bode-api]]` — middleware extrai AuthUser
- `[[capra-analytics]]` — adapter consulta `/api/v2/me` para resolver permissions
- `[[Auth Flow JWKS]]` — fluxo de validação JWT que precede a cascade
