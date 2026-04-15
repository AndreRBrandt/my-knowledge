---
type: concept
status: stable
domain: organization
project: [bode-api, capra-analytics, capra-dbt]
tags: [domain, organization, rbac]
created: 2026-04-14
related: "[[Empresa]], [[Roles]], [[RBAC Cascade Model]]"
---

# Filial

> Unidade operacional individual (uma loja física). Nível mais granular da hierarquia organizacional do grupo.

## O que é
Representa uma loja física. Toda venda, despesa, escala de RH e métrica está atrelada a uma filial. É a unidade pela qual permissões e dashboards são particionados.

## Atributos (canonical em `bode-api`)
```sql
CREATE TABLE filiais (
    id         SERIAL PRIMARY KEY,
    name       TEXT NOT NULL,
    active     BOOLEAN NOT NULL DEFAULT true,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

- **id SERIAL** — facilita admin panel; FK em `user_filiais`, dados operacionais
- **name** — display name (`Bode do Nô - Boa Viagem`)
- **active** — soft delete; filial inativa não desaparece dos relatórios históricos

## Hierarquia
```
Empresa (marca: Bode do Nô / Burguer do Nô)
  └── Filial (loja física: Boa Viagem, Afogados, Olinda, ...)
        └── Operações (vendas, RH, etc.)
```

## Relações com RBAC
- `[[Roles]]` `gestor_filial` deve ser restringido a um subconjunto de filiais via `user_filiais`
- `[[Roles]]` `adm`/`diretor`: SEM linha em `user_filiais` = vê TODAS (regra explícita no seed)
- Sistema OLTP (auth/RBAC) usa `id INT`; correlação com OLAP (dbt) feita via UUID do usuário

## Onde é usado
- **`[[bode-api]]`** — tabela canônica `filiais` + `user_filiais` (RBAC)
- **`[[capra-dbt]]`** — `dim_filial` (gold), JOINs em todos os obts (vendas, financeiro, RH)
- **`[[capra-analytics]]`** — filtro principal de todo dashboard (`FILTER_IDS.filial`)
- Adapter de filtros mapeia: cubo Vendas usa `[Loja]`, Auditoria `[Unidade]`, Financeiro `[Filial]` (ver `[[Filter Registry]]`)

## Glossary
"Loja" e "Unidade" são sinônimos de "Filial" no contexto operacional. Veja `[[Glossary (consolidated)]]`.
