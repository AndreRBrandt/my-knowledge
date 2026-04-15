---
type: architecture
status: living
project: [bode-api]
tags: [testing, ci-cd, coverage, tarpaulin]
created: 2026-04-15
related: "[[bode-api]], [[ADR-BODE-007 CI CD Quality Gates]], [[Crate Boundaries]]"
---

# Two-pipeline Testing Strategy

> Estratégia de dois pipelines de teste: **unit no CI** (todo PR, sem infra externa) + **integration no homolog** (pós-deploy, com serviços reais). Cada código é testado na camada certa.

## Problema

Coverage 100% no CI é impossível sem ignorar realidade:
- Código que fala com **Postgres real** — não testa sem DB
- Código que valida **JWT real do Supabase** — não testa sem HTTP
- Código de **startup** (init tracing, DB pool, auth provider) — tipicamente excluído

Soluções populares falham:
- **Mock tudo:** coverage sobe mas testa o mock, não o código
- **Rodar DB em cada PR:** lento, complexo, pode ficar flaky
- **Aceitar coverage baixo:** perde quality gate

## Solução: dois pipelines

```
┌───────────────────────────────────────────────────────────┐
│ PIPELINE 1: CI (todo PR)                                   │
│ ─────────────────────────                                  │
│ - Unit tests (código puro, sem infra)                      │
│ - Coverage ≥ 90% (fail CI se menor)                        │
│ - Rápido (<2min)                                           │
│ - Exclusões cirúrgicas: #[cfg(not(tarpaulin_include))]    │
└───────────────────────────────────────────────────────────┘
                        ↓ merge main
┌───────────────────────────────────────────────────────────┐
│ PIPELINE 2: HOMOLOG (pós-deploy)                           │
│ ─────────────────────────────                              │
│ - Integration tests (DB real, auth real, HTTP real)        │
│ - Smoke tests (health, auth flow, endpoints críticos)      │
│ - Cobre o que CI excluiu                                   │
│ - Se falhar: rollback automático                           │
└───────────────────────────────────────────────────────────┘
```

## O que é testado onde

| Código | Testado onde | Como |
|--------|--------------|------|
| Lógica de domínio (bode-core) | CI | Unit tests puros |
| SQL queries (bode-db) | Homolog | Testado contra DB real |
| Auth middleware | CI (erro) + Homolog (sucesso) | Mock em CI, real em homolog |
| Handlers simples | CI | Unit com state mock |
| Startup (main.rs) | Homolog (processo sobe?) | Excluído de coverage |
| Integração Supabase JWKS | Homolog | HTTP real |

## Exclusões cirúrgicas

Pra código que SÓ faz sentido em integração, usar attribute tarpaulin:

```rust
#[cfg(not(tarpaulin_include))]
pub async fn load_auth_user(pool: &PgPool, user_id: &str) -> Result<AuthUser, AppError> {
    // Código que precisa de DB real — testado em homolog
}
```

### Quando excluir

- **Startup orchestration:** `main()`, `init_database()`, `init_tracing()`
- **Connection factories:** `create_pool()`
- **External HTTP calls:** `SupabaseAuthProvider::new()` (fetch JWKS)
- **Integration-only queries:** `load_auth_user()` e similares

### Quando NÃO excluir

- Pure logic, mesmo que ligada a módulo de integração
- Validações, transformações, error mapping
- Extractors que podem ser testados com mocks

## Configuração CI (bode-api)

```yaml
# .github/workflows/ci.yml (excerpt)
coverage:
  name: Coverage
  needs: [fmt, clippy, test]
  steps:
    - uses: actions/checkout@v6
    - run: cargo tarpaulin --workspace --out xml --fail-under 90 --skip-clean
```

`--fail-under 90` fail CI se coverage < 90% (considerando código NÃO excluído).

## Resultado no bode-api

**Estado atual (Fase 1 completa):**
- 49 testes unitários
- 92.54% coverage (124/134 linhas testáveis)
- Código excluído com `#[cfg(not(tarpaulin_include))]`:
  - `main.rs` (startup)
  - `auth/supabase.rs` (HTTP real)
  - `repositories/users.rs` (DB real)
  - `bode-db/lib.rs::create_pool` (DB real)
  - `routes/me.rs` (precisa auth + DB)

**Pendente (Fase 2):** Integration + smoke tests em homolog pra cobrir o que CI excluiu.

## Técnica: fast-fail pool nos testes

Pra testar path de erro de `/health/db` sem DB real, cria pool lazy que falha rápido:

```rust
// tests/common/mod.rs
let pool = PgPoolOptions::new()
    .acquire_timeout(Duration::from_secs(1))  // 1s em vez de 30s default
    .connect_lazy("postgresql://test:test@127.0.0.1:1/test")
    .expect("lazy pool should not actually connect");
```

Porta 1 = connection refused imediato. Teste roda em ~1s em vez de 30s.

## Limitações aceitas (tech debt)

3 linhas no coverage atual são **limitações do tarpaulin** (código É executado mas não contabilizado):
- `error.rs:40-41` — tracing macro expansion
- `config.rs:118` — inlined branch do Err

Documentado em TD-003 do projeto.

## Referências

- `[[bode-api]]` — projeto
- `[[ADR-BODE-007 CI CD Quality Gates]]` — regras do CI
- `[[Crate Boundaries]]` — separação que facilita testabilidade
- [cargo-tarpaulin](https://github.com/xd009642/tarpaulin) — ferramenta de coverage
