---
type: architecture
status: living
project: [bode-api]
tags: [rust, architecture, crates, separation-of-concerns]
created: 2026-04-15
related: "[[bode-api]], [[ADR-BODE-001 Rust Backend Language]]"
---

# Crate Boundaries

> Regras STRICT de separação entre crates do workspace `bode-api`. Violar quebra testabilidade e portabilidade.

## Estrutura

```
bode-api/
├── Cargo.toml  (virtual workspace)
└── crates/
    ├── bode-core/   ← domínio puro
    ├── bode-db/     ← persistência
    ├── bode-server/ ← HTTP/API
    └── bode-sync/   ← ETL
```

## Regras por crate

### bode-core
- **O que tem:** domain types, business rules, pure logic, traits (interfaces)
- **Pode importar:** `serde`, `thiserror`, `chrono`, `uuid`, `std`
- **NÃO pode importar:** `sqlx`, `reqwest`, `tokio`, `axum`, qualquer lib de I/O
- **Regra de ouro:** se a feature requer rede/filesystem/DB, NÃO VAI AQUI

### bode-db
- **O que tem:** implementações de repositórios (traits de bode-core), queries SQL, pool management
- **Pode importar:** `bode-core`, `sqlx`, `tokio`
- **NÃO pode importar:** `axum` (não tem handlers aqui)

### bode-server
- **O que tem:** Axum routes, middleware, handlers, HTTP-specific types, auth implementations
- **Pode importar:** `bode-core`, `bode-db`, `bode-sync`, `axum`, `tower`, `jsonwebtoken`
- **Regra:** handlers são FINOS — lógica vive em bode-core, queries em bode-db

### bode-sync
- **O que tem:** ETL scripts, extractors (HTTP), transformers, loaders
- **Pode importar:** `bode-core`, `bode-db`, `reqwest`, `tokio`
- **NÃO pode importar:** `axum` (ETL não é HTTP server)

## Visualização

```
                    ┌──────────────┐
                    │  bode-core   │  ZERO I/O
                    │  (traits,    │
                    │   types)     │
                    └──────┬───────┘
                           │ depende
          ┌────────────────┼────────────────┐
          │                │                │
          ↓                ↓                ↓
    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │ bode-db  │    │bode-sync │    │bode-server│
    │ (sqlx)   │    │(reqwest) │    │  (axum)  │
    └──────────┘    └─────┬────┘    └────┬─────┘
                          │              │
                          └──────────────┤
                                         ↓
                                   ┌──────────┐
                                   │bode-server│  bin final
                                   │ (binary)  │
                                   └──────────┘
```

## Anti-patterns

### ❌ Business logic em bode-server
```rust
// ERRADO — handler faz lógica
async fn calcular_desconto(Json(venda): Json<Venda>) -> Json<f64> {
    let desc = if venda.valor > 100.0 { 0.1 } else { 0.0 };
    Json(desc)
}

// CERTO — handler fino, lógica em bode-core
async fn calcular_desconto(Json(venda): Json<Venda>) -> Json<f64> {
    Json(bode_core::desconto::calcular(&venda))
}
```

### ❌ I/O em bode-core
```rust
// ERRADO — bode-core puxando reqwest
use reqwest::Client;  // bode-core/Cargo.toml NÃO deveria ter reqwest
async fn fetch_exchange_rate() -> f64 { ... }

// CERTO — trait em bode-core, impl em outro crate
pub trait ExchangeRateProvider: Send + Sync {
    async fn get(&self, from: &str, to: &str) -> Result<f64, CoreError>;
}
// Implementação concreta vai em bode-sync ou outro crate
```

### ❌ SQL em bode-server
```rust
// ERRADO — handler fazendo query direto
async fn list_users(State(pool): State<PgPool>) {
    sqlx::query("SELECT * FROM users").fetch_all(&pool).await
}

// CERTO — handler chama repository
async fn list_users(State(pool): State<PgPool>) {
    bode_db::users::list_all(&pool).await
}
```

## Por que isso importa

1. **Testabilidade:** bode-core testa com mocks, sem subir DB ou mock HTTP
2. **Portabilidade:** trocar Postgres por outro DB = reescrever bode-db, bode-core não muda
3. **Clareza:** olhando o Cargo.toml de um crate, sabe o que ele faz
4. **Compile time:** separation reduz recompilação em mudanças
5. **Segurança:** lógica de negócio isolada de superfície de ataque (HTTP)

## Enforcement

- **Via Cargo.toml:** cada crate declara apenas deps permitidas
- **Via clippy:** `#![warn(clippy::disallowed_methods)]` pra apis proibidas
- **Via review:** PRs que violam são rejeitadas
- **Via ADR:** mudanças nas regras viram ADR

## Referências

- `[[bode-api]]` — projeto
- `[[ADR-BODE-001 Rust Backend Language]]` — decisão de usar Rust
- [Onion Architecture](https://alistair.cockburn.us/hexagonal-architecture/) — princípio geral
