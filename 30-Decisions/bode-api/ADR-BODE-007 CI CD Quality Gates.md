---
type: decision
status: stable
project: [bode-api]
tags: [adr, ci-cd, quality, testing]
created: 2026-04-07
related: "[[ADR-BODE-006 Three Environments]]"
---

# ADR-BODE-007 Quality Gates in CI/CD

**Status:** Accepted
**Date:** 2026-04-07

## Context
Code quality must be enforced automatically. Manual review alone is insufficient — the CI pipeline must block merges that don't meet quality standards.

## Decision
Every PR must pass these quality gates before merge:

1. **`cargo fmt --check`** — Code formatting (zero tolerance)
2. **`cargo clippy -- -D warnings`** — Lint (all warnings are errors)
3. **`cargo test`** — All tests pass
4. **`cargo build --release`** — Release build succeeds
5. **Coverage threshold** — Minimum **90%** line coverage (enforced via cargo-tarpaulin)
6. **Docker build** — Multi-stage build succeeds (on PRs only, dry-run)

If ANY gate fails, the PR cannot be merged.

> **Nota:** ADR original dizia 70% coverage. Em 2026-04-14 a regra foi elevada pra 90% com exclusão cirúrgica de código de integração via `#[cfg(not(tarpaulin_include))]`. Ver `[[Two-pipeline Testing Strategy]]`.

## Consequences

### Positive
- Consistent code style across all contributors
- Common Rust pitfalls caught by clippy before review
- No broken builds reach main branch
- Coverage threshold prevents untested code from shipping
- Release build verification catches optimization-specific issues

### Negative
- CI pipeline takes ~10-15 minutes (Rust compile times)
- Developers must run checks locally to avoid CI failures
- Coverage threshold may need adjustment as codebase grows

### Risks
- Flaky tests could block legitimate PRs (mitigated by retry policy)
- Overly strict clippy lints may require `#[allow]` annotations

## Alternatives Considered

| Alternative | Why rejected |
|-------------|--------------|
| No CI checks | Unacceptable for production financial data platform |
| Only tests (no clippy/fmt) | Misses preventable bugs and inconsistent formatting |
| Pre-commit hooks only | Can be bypassed with --no-verify |

## References
- `[[bode-api]]` — project hub
- `[[Two-pipeline Testing Strategy]]` — unit CI + integration homolog
