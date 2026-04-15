---
type: decision
status: stable
project: [bode-api]
tags: [adr, docker, deployment, build]
created: 2026-04-07
related: "[[ADR-BODE-004 Blue-Green Deployment]], [[Multi-stage Docker Build]]"
---

# ADR-BODE-003 Docker Multi-Stage Build

**Status:** Accepted
**Date:** 2026-04-07

## Context
We need a containerization strategy that produces small, secure images for deployment while maintaining fast CI builds.

## Decision
Use Docker multi-stage builds:
1. **Builder stage:** Rust official image, compile with `--release` (LTO + strip)
2. **Runtime stage:** `debian:bookworm-slim` with only the static binary + CA certs

Target image size: < 30 MB.

## Consequences

### Positive
- Minimal attack surface (no compiler, no source code in production image)
- Small image size reduces pull times and storage costs
- Reproducible builds across environments
- LTO + strip produces optimized, small binaries

### Negative
- Build time is longer due to full release compilation (~5-10 min)
- Docker layer caching for Cargo dependencies requires careful Dockerfile structuring

### Risks
- Cross-compilation may be needed if CI runners differ from deployment architecture

## Alternatives Considered

| Alternative | Why rejected |
|-------------|--------------|
| Single-stage build | Large image (>1GB), includes compiler and source |
| Distroless | Harder to debug, less tooling for PostgreSQL SSL deps |
| Alpine-based | musl libc issues with some Rust crates |

## References
- `[[bode-api]]` — project hub
- `[[Multi-stage Docker Build]]` — pattern details
