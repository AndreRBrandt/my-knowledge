---
type: decision
status: stable
project: [bode-api]
tags: [adr, environments, deployment]
created: 2026-04-07
related: "[[ADR-BODE-004 Blue-Green Deployment]], [[ADR-BODE-007 CI CD Quality Gates]]"
---

# ADR-BODE-006 Three Environments (Dev / Homolog / Prod)

**Status:** Accepted
**Date:** 2026-04-07

## Context
We need isolated environments for development, testing, and production to prevent untested code from reaching end users.

## Decision
Three environments with clear promotion gates:

| Environment | Trigger | Database | URL |
|-------------|---------|----------|-----|
| **dev** | Local `cargo run` | Local PG or Supabase dev project | localhost:3000 |
| **homolog** | Merge to `main` (auto-deploy) | Supabase staging project | bi-homolog.bodedono.com.br |
| **prod** | Git tag `v*` (auto-deploy) | Supabase production project | bi.bodedono.com.br |

Each environment has its own `.env` with isolated database URLs and API keys.

## Consequences

### Positive
- Clear separation prevents accidental production data corruption
- Homolog acts as final validation before production
- Automatic deployment reduces manual error
- Each environment can be independently scaled

### Negative
- Three sets of environment variables to maintain
- Supabase cost for staging project
- Must keep staging database schema in sync with production

### Risks
- Schema drift between environments if migrations aren't applied consistently

## Alternatives Considered

| Alternative | Why rejected |
|-------------|--------------|
| Two environments (dev + prod) | No staging validation, higher risk |
| Feature flags instead of environments | Adds runtime complexity, doesn't solve data isolation |

## References
- `[[bode-api]]` — project hub
