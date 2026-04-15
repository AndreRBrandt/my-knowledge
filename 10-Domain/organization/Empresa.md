---
type: concept
status: stable
domain: organization
project: [bode-api, capra-analytics]
tags: [domain, organization, brand]
created: 2026-04-14
related: "[[Filial]]"
---

# Empresa (Marca)

> Agrupamento por marca/CNPJ. Uma empresa contém várias `[[Filial|Filiais]]` da mesma identidade comercial.

## O que é
Conceito de marca/grupo no negócio do Andre. Atualmente:

| Empresa | Filiais |
|---------|---------|
| Bode do Nô | Boa Viagem, Afogados, Olinda, Guararapes, Tacaruna |
| Burguer do Nô | Rio Mar, Guararapes, Boa Viagem |

## Status no schema atual
**Nota:** Hoje (2026-04-15) a tabela `filiais` no `bode-api` NÃO tem coluna explícita de empresa. A marca é inferida pelo nome (`Bode do Nô - X` vs `Burguer do Nô - Y`). Isso funciona enquanto há 2 marcas com prefixos consistentes, mas é fragilidade.

**Próximo passo natural:** introduzir tabela `empresas` com FK em `filiais.empresa_id`. Justifica-se quando:
- Roles passarem a ser por empresa (`gestor_empresa` que vê todas as filiais de uma marca)
- Dashboards forem filtráveis por marca
- Onboarding de novas marcas tornar o "parsing por nome" inviável

## Onde é usado
- **`[[capra-analytics]]`** — alguns dashboards filtram via prefixo do nome (frágil)
- **`[[capra-dbt]]`** — `dim_filial` poderia derivar marca via CASE no staging

## Edge case
Quando uma marca encerra ou muda nome, a "inferência por prefixo" quebra retroativo. Mover para coluna explícita resolve.
