---
type: architecture
status: living
project: [bode-api]
tags: [logging, observability, tracing, json]
created: 2026-04-15
related: "[[bode-api]], [[Request ID Propagation]], [[ADR-BODE-010 Observability Metrics]]"
---

# Structured Logging

> Logs em formato estruturado (JSON) em produção, legível (human-readable) em dev. Cada linha de log é um evento parseable com contexto rico.

## Por que estruturado

**Não estruturado (texto):**
```
2026-04-14 18:45:01 INFO request handled in 45ms
```
- Difícil parsear
- Dificil filtrar ("quantas requests em /api/v2/me demoraram > 100ms?")
- Perde contexto (qual user? qual request id?)

**Estruturado (JSON):**
```json
{
  "timestamp": "2026-04-14T18:45:01Z",
  "level": "INFO",
  "target": "bode_server",
  "message": "request handled",
  "request_id": "abc-123",
  "path": "/api/v2/me",
  "method": "GET",
  "status": 200,
  "duration_ms": 45,
  "user_id": "user-456"
}
```
- Parseable por ferramentas (Grafana Loki, ElasticSearch, etc)
- Filtrável por qualquer campo
- Correlação via `request_id`
- Agregação trivial

## Implementação (bode-api)

### Stack
- **Crate:** `tracing` + `tracing-subscriber` (features: `env-filter`, `json`)
- **Formato:** JSON em produção, human em dev

### Switch condicional

```rust
// main.rs
fn init_tracing(config: &ServerConfig) {
    let env_filter = tracing_subscriber::EnvFilter::try_from_default_env()
        .unwrap_or_else(|_| "bode_server=debug,tower_http=debug".into());

    if config.environment.is_production_like() {
        // JSON structured (homolog/prod)
        tracing_subscriber::registry()
            .with(env_filter)
            .with(tracing_subscriber::fmt::layer().json())
            .init();
    } else {
        // Human-readable (dev)
        tracing_subscriber::registry()
            .with(env_filter)
            .with(tracing_subscriber::fmt::layer())
            .init();
    }
}
```

### Env var switch
`ENVIRONMENT=development` → human
`ENVIRONMENT=homolog` ou `production` → JSON

### Uso nos handlers

```rust
use tracing::{info, warn, error, debug};

async fn handler() {
    info!("request received");                          // simples
    
    info!(user_id = %user.id, action = "login", "user logged in");  // estruturado
    
    if let Err(e) = operation() {
        error!(error = %e, "operation failed");          // erro com contexto
    }
}
```

Os campos viram colunas no JSON. `%` formata via Display, `?` via Debug.

## Níveis de log

| Nível | Uso |
|-------|-----|
| `error!` | Falhas que afetam request (DB down, panic, bug crítico) |
| `warn!` | Situação anormal mas não falha (deprecated API usada, retry bem-sucedido) |
| `info!` | Eventos normais de negócio (request servido, user logado, job completado) |
| `debug!` | Detalhes pra debugging (parâmetros, decisões internas) |
| `trace!` | Altíssima frequência (loop iterations, etc) — raramente ativar |

## Filtros via env

```bash
# Só bode_server em info, tower em warn
RUST_LOG=bode_server=info,tower_http=warn cargo run

# Tudo em debug (verbose)
RUST_LOG=debug cargo run

# Específico: só queries SQL
RUST_LOG=sqlx=debug cargo run
```

## Logs sensíveis

NUNCA logar:
- Tokens (JWT, refresh, session)
- Senhas
- PII sensível (CPF completo, cartão)

Ao logar user, use ID, nunca email/CPF direto em produção.

## Observabilidade em produção

Com JSON logs + Grafana Loki / Promtail:

```
Query Loki:
{app="bode-api"} |= "error" | json | status >= 500 | request_id="abc-123"
```

Acha todo erro 5xx com request_id específico em segundos.

## Relação com request_id

Ver `[[Request ID Propagation]]`. Cada request ganha UUID único que aparece em TODO log daquele request → correlação completa.

## Relação com métricas

Logs são **eventos** (algo aconteceu). Métricas são **séries** (quantos aconteceram).

Exemplo:
- **Log:** `{"message": "request handled", "duration_ms": 45}`
- **Métrica:** `http_request_duration_ms{path="/api/v2/me",status="200"}`

Ver `[[ADR-BODE-010 Observability Metrics]]` pra métricas.

## Referências

- `[[bode-api]]` — projeto
- `[[Request ID Propagation]]` — correlação entre logs
- `[[ADR-BODE-010 Observability Metrics]]` — complementar
- [tracing crate docs](https://docs.rs/tracing/)
