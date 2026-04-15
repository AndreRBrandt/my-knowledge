---
type: decision
status: stable
project: [bode-api]
tags: [adr, database, supabase, postgres, vendor-lockin]
created: 2026-04-07
related: "[[ADR-BODE-009 Zero Vendor Lock-in]], [[dbt Architecture]]"
---

# ADR-BODE-005 Database Abstraction (Supabase Now, Own PG Later)

**Status:** Accepted
**Date:** 2026-04-07

## Context
The platform currently uses Supabase (managed PostgreSQL) for all data storage. Long-term, we may want to run our own PostgreSQL for cost and control reasons. The Rust backend must work with both without code changes.

## Decision
Use sqlx with raw SQL queries targeting standard PostgreSQL. No Supabase-specific client features in the data layer. Connection is configured via `DATABASE_URL` env var — switching from Supabase to self-hosted PG requires only changing this URL.

Supabase-specific features (Auth, RLS) are handled at the edge/middleware level, not in repository code.

## Consequences

### Positive
- Zero vendor lock-in at the database layer
- Standard PostgreSQL SQL works everywhere
- Migration path is a DNS/connection string change
- sqlx compile-time checks work against any PostgreSQL instance

### Negative
- Cannot use Supabase JS client convenience features (realtime, storage client)
- Must implement auth token verification manually
- RLS policies are database-level (work with any client)

### Risks
- Some Supabase-managed features (automatic backups, dashboard) lost on migration

## Alternatives Considered

| Alternative | Why rejected |
|-------------|--------------|
| Supabase client SDK | Rust SDK immature, creates vendor lock-in |
| ORM (Diesel/SeaORM) | Adds abstraction without solving the actual portability need |
| Multi-database support | Over-engineering; PostgreSQL is the only target |

## References
- `[[bode-api]]` — project hub
- `[[ADR-BODE-009 Zero Vendor Lock-in]]` — strategy extends to all externals
- `[[dbt Architecture]]` — same DB target as dbt pipeline
