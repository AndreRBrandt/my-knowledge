---
type: architecture
status: living
project: [bode-api]
tags: [docker, build, deployment]
created: 2026-04-15
related: "[[bode-api]], [[ADR-BODE-003 Docker Multi-stage Build]]"
---

# Multi-stage Docker Build

> Pattern de build Docker em 2+ estágios: stage de build (com toolchain) produz artefato; stage runtime (mínimo) copia só o artefato. Resultado: imagens pequenas, seguras, rápidas.

## Por que usar

Build de Rust precisa:
- Compilador (~1GB de toolchain)
- Cargo, dependências, código fonte
- Tempo de compilação (~5-10min release)

Runtime precisa:
- Binário estático (~15MB stripped)
- CA certs pra HTTPS
- Usuário non-root
- Não precisa de NADA do build

**Single-stage:** imagem final ~1GB+ (inclui toolchain + fonte + deps de build)
**Multi-stage:** imagem final ~95MB (só debian-slim + binário)

## Estrutura (bode-api)

```dockerfile
# ============================================
# Stage 1: Builder
# ============================================
FROM rust:1.88-bookworm AS builder
WORKDIR /build

# Cache de deps: copia só manifestos primeiro
COPY Cargo.toml Cargo.lock ./
COPY crates/*/Cargo.toml crates/*/

# Dummy files pra compilar só deps (cache layer)
RUN mkdir -p crates/*/src && echo "pub fn _dummy() {}" > ...
RUN cargo build --release --bin bode-server || true

# Agora copia source real + recompila
RUN rm -rf crates/*/src
COPY crates/ crates/
COPY migrations/ migrations/

# Touch força recompilação do NOSSO código (não deps)
RUN touch crates/*/src/*.rs
RUN cargo build --release --bin bode-server

# ============================================
# Stage 2: Runtime
# ============================================
FROM debian:bookworm-slim AS runtime

RUN apt-get update && \
    apt-get install -y --no-install-recommends ca-certificates tini wget && \
    rm -rf /var/lib/apt/lists/*

RUN groupadd --gid 1001 appgroup && \
    useradd --uid 1001 --gid appgroup --shell /bin/false appuser

COPY --from=builder /build/target/release/bode-server /usr/local/bin/bode-server
RUN chown appuser:appgroup /usr/local/bin/bode-server

USER appuser
EXPOSE 8000

HEALTHCHECK --interval=10s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8000/health || exit 1

ENTRYPOINT ["tini", "--"]
CMD ["bode-server"]
```

## Truques importantes

### 1. Cache de dependências
Copiar **Cargo.toml + Cargo.lock PRIMEIRO** e criar dummy source files. Isso cacheia a compilação de dependências.

Quando só o código muda, Docker reaproveita o layer das deps (economiza 3-5min no CI).

### 2. Touch após copy
`COPY crates/ crates/` pode não trigar recompilação se Cargo julga que nada mudou nos sources "dummy". Solução: `touch` em todos os lib.rs/main.rs pra forçar.

### 3. Release profile no Cargo.toml
```toml
[profile.release]
lto = true           # Link-time optimization
codegen-units = 1    # Single codegen = código mais otimizado
strip = true         # Remove símbolos de debug
```

### 4. Runtime deps mínimas
Só `ca-certificates` (HTTPS), `tini` (PID 1), `wget` (healthcheck).
Nada de curl, bash fancy, etc.

### 5. Tini como PID 1
Tini é init minimal. Reap zombies + forward signals (SIGTERM → binary).
Sem tini, `docker stop` pode demorar 10s + SIGKILL = perda de requests em flight.

### 6. Non-root user
Container roda como `appuser:1001`, não root. Mitiga impacto de RCE.

## Trade-offs

| Aspecto | Multi-stage | Single-stage |
|---------|-------------|--------------|
| Tamanho final | ~95MB | ~1GB |
| Attack surface | Mínima | Inclui toolchain |
| Build time primeiro build | +0 (igual) | +0 |
| Build time com cache | Bom (cache de deps) | Ruim (recompila tudo) |
| Complexidade Dockerfile | Maior | Menor |

## Gotchas

### MSRV mismatch
`rust:1.82-bookworm` pode ter libs mais antigas que o `edition2024` requerido por algumas crates. Use imagem >= versão mínima do Cargo.lock.

**No caso do bode-api:** precisou subir de `rust:1.82` → `rust:1.88` por causa de `home@0.5.12` que requer 1.88.

### `--no-cache` em CI
Em CI que roda no self-hosted, layers podem ficar stale entre versões. Usar `docker build --no-cache` garante build fresh (mais lento mas previsível).

## Referências

- `[[bode-api]]` — projeto
- `[[ADR-BODE-003 Docker Multi-stage Build]]` — decisão
- [Docker Best Practices — multi-stage](https://docs.docker.com/build/building/multi-stage/)
