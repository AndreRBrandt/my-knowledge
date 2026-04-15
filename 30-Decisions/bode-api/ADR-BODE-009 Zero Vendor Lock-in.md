---
type: decision
status: stable
project: [bode-api]
tags: [adr, vendor-lockin, architecture, traits, strategy]
created: 2026-04-09
related: "[[ADR-BODE-005 Database Abstraction]], [[Auth Flow JWKS]]"
---

# ADR-BODE-009 Zero Vendor Lock-in — Abstração em todas as camadas

**Status:** Accepted
**Date:** 2026-04-09

## Context
O sistema hoje depende de Supabase para: banco (PostgreSQL), autenticação (Supabase Auth + JWT), storage (buckets), e algumas RPCs. O plano é migrar para PostgreSQL self-hosted (primeiro na VPS, depois DO Managed Database). Essa migração não pode exigir reescrever o backend.

Além disso, queremos que o sistema seja portável — trocar qualquer componente externo deve ser uma mudança de configuração + implementação de interface, nunca refatoração de lógica.

## Decision
**Toda dependência externa deve ser acessada via trait (interface) no `bode-core`, com implementação concreta em outro crate.** O código de negócio nunca sabe qual provider está por trás.

### Camadas de abstração

| Camada | Trait (bode-core) | Implementação atual | Futura |
|--------|-------------------|---------------------|--------|
| Database | Pool via `DATABASE_URL` | Supabase PostgreSQL | DO Managed / VPS PG |
| Auth | `AuthProvider` trait | Supabase JWT (HS256) | Próprio (JWT + bcrypt) |
| User store | `UserRepository` trait | Supabase auth.users | Tabela própria users |
| Storage | `StorageProvider` trait | Cloudflare R2 | S3 / MinIO |
| Cache | `CacheProvider` trait | Redis | Redis (mantém) |
| Notifications | `NotificationSender` trait | Teams webhook | Teams / Email / Push |

### Regras

1. **bode-core** define traits e tipos. NUNCA importa libs de providers.
2. **bode-db** implementa traits de repositório usando sqlx (SQL padrão, sem extensões Supabase).
3. **bode-server** implementa traits de auth e faz a injeção de dependência.
4. **Nenhum código** deve referenciar "supabase" fora do módulo de implementação concreta.
5. **Lock-in aceito** deve ser documentado com ADR próprio justificando o tradeoff.

### Auth: plano de migração

**Fase atual:** Supabase Auth emite JWT → bode-api valida com JWKS (ES256)
- Lock-in: user store em `auth.users` do Supabase
- JWT validation via JWKS → padrão, funciona com qualquer emissor

**Fase futura (pós-migração DB):**
- Criar tabela `users` própria no PostgreSQL
- Implementar login/signup no bode-api (bcrypt + JWT próprio)
- Frontend troca `supabase.auth.signIn()` por `fetch('/api/v2/auth/login')`
- Migração de users: export Supabase → import tabela própria

**Transição:** O backend aceita AMBOS os JWTs durante a migração (Supabase e próprio) via campo `iss` (issuer) no token.

## Consequences

### Positive
- Trocar Supabase por PG self-hosted = mudar `DATABASE_URL`
- Trocar auth provider = implementar nova struct que satisfaz `AuthProvider` trait
- Trocar storage = implementar nova struct que satisfaz `StorageProvider` trait
- Código de negócio (bode-core) nunca muda por causa de provider
- Testável: mocks implementam os mesmos traits

### Negative
- Mais código upfront (traits + implementações)
- Indireção extra (trait dispatch vs chamada direta)
- Auth próprio é mais trabalho que usar Supabase Auth

### Lock-ins aceitos (documentados)
- **PostgreSQL** como engine de banco (não vamos suportar MySQL/SQLite) — tradeoff positivo: sqlx compile-time checks
- **Redis** como cache — padrão de mercado, sem alternativa melhor pra esse uso

## Alternatives Considered

| Alternative | Why rejected |
|-------------|--------------|
| Manter Supabase Auth pra sempre | Custo, dependência, limitação de customização |
| Multi-database (PG + SQLite + MySQL) | Over-engineering, só vamos usar PostgreSQL |
| Abstrair Redis também | Redis é commodity, trocar por outro cache in-memory não justifica abstração |

## References
- `[[bode-api]]` — project hub
- `[[ADR-BODE-005 Database Abstraction]]` — específico pra DB
- `[[Auth Flow JWKS]]` — implementação atual de auth
