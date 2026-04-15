---
type: decision
status: stable
project: [bode-api]
tags: [adr, rust, stack, axum, sqlx]
created: 2026-04-07
related: "[[ADR-BODE-001 Rust Backend Language]], [[ADR-BODE-005 Database Abstraction]]"
---

# ADR-BODE-002 Axum + sqlx + serde Stack

**Status:** Accepted
**Date:** 2026-04-07

## Context
Having chosen Rust (`[[ADR-BODE-001 Rust Backend Language]]`), we need to select the web framework, database driver, and serialization library. These form the core stack that everything else builds on.

## Decision
- **HTTP:** Axum 0.8 — built on tower/hyper, modular middleware, strong typing for extractors
- **Database:** sqlx 0.8 — compile-time checked SQL, async, direct PostgreSQL support
- **Serialization:** serde + serde_json — de facto standard, derive macros, zero-copy deserialization
- **Async runtime:** tokio — industry standard, required by Axum and sqlx
- **Error handling:** thiserror (library crates) + anyhow (binary crate)

## Consequences

### Positive
- Axum's extractor pattern enforces type-safe request parsing
- sqlx compile-time query checking catches SQL errors before runtime
- Tower middleware ecosystem (rate limiting, cors, tracing) works out of the box
- All three libraries are mature, well-maintained, and widely adopted

### Negative
- sqlx compile-time checks require a live database during development
- Axum's type-level routing can produce complex error messages

### Risks
- Major version bumps in Axum could require migration effort

## Alternatives Considered

| Alternative | Why rejected |
|-------------|--------------|
| Actix-web | Less idiomatic, actor model adds complexity we don't need |
| Diesel | ORM overhead, less flexibility for complex analytics queries |
| SeaORM | Additional abstraction layer over sqlx, unnecessary complexity |

## References
- `[[bode-api]]` — project hub
