---
type: decision
status: stable
project: [bode-api]
tags: [adr, rust, backend, migration]
created: 2026-04-07
related: "[[ADR-BODE-002 Axum sqlx serde Stack]], [[Strangler Fig Migration]]"
---

# ADR-BODE-001 Rust as Backend Language

**Status:** Accepted
**Date:** 2026-04-07

## Context
The existing backend (`bode-analytics-api`) is TypeScript running on Node.js (Hono framework). While functional, it has limitations: high memory usage under load, lack of compile-time safety for complex data pipelines, and runtime type errors that reach production despite TypeScript. The platform handles financial data, HR data, and sales analytics for multiple restaurant units — correctness is critical.

## Decision
Rewrite the backend in Rust using a Cargo workspace with clearly separated crates. Rust provides memory safety without GC, zero-cost abstractions, and compile-time guarantees that eliminate entire classes of bugs.

## Consequences

### Positive
- Compile-time safety eliminates null/undefined errors
- Predictable performance (no GC pauses)
- Lower memory footprint (~10x less than Node.js for equivalent workloads)
- Strong type system catches data pipeline errors at compile time
- Single static binary simplifies deployment

### Negative
- Steeper learning curve for team members
- Longer compile times than TypeScript
- Smaller ecosystem for some niche libraries
- Initial development velocity may be slower

### Risks
- Team velocity may dip during ramp-up period
- Some Supabase client features may need manual implementation

## Alternatives Considered

| Alternative | Why rejected |
|-------------|--------------|
| Keep TypeScript/Node.js | Runtime errors, memory issues, insufficient safety for financial data |
| Go | Good performance but weaker type system, no sum types, less expressive error handling |
| Java/Kotlin (Spring) | Heavy runtime, slow startup, over-engineered for our scale |

## References
- `[[bode-api]]` — project hub
- `[[Strangler Fig Migration]]` — how we replace the TS backend incrementally
