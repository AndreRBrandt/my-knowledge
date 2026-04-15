---
type: architecture
status: living
project: [bode-api]
tags: [observability, tracing, request-id, debugging]
created: 2026-04-15
related: "[[bode-api]], [[Structured Logging]]"
---

# Request ID Propagation

> Cada request HTTP ganha um UUID único via header `x-request-id`. O ID é propagado em todo log daquele request. Resultado: correlação completa pra debugar — um ID leva a toda a história.

## Fluxo

```
Cliente                    Traefik           bode-api                  Logs
  │                          │                  │                        │
  │── GET /api/v2/me ───────▶│                  │                        │
  │                          │── x-req-id:  ───▶│                        │
  │                          │   generated      │ Middleware injeta       │
  │                          │                  │ no tracing span         │
  │                          │                  │                        │
  │                          │                  │ handler executa        │
  │                          │                  │  tracing::info!() ────▶│ {"request_id": "abc-123", ...}
  │                          │                  │  load_user() ─────────▶│ {"request_id": "abc-123", ...}
  │                          │                  │  query() ─────────────▶│ {"request_id": "abc-123", ...}
  │                          │                  │                        │
  │                          │◀── response  ────│                        │
  │◀── 200 + x-req-id:abc ──│                  │                        │
```

Agora tu tem:
- Cliente vê o `x-request-id` no response
- Se algo deu errado, manda pra suporte
- Suporte busca `abc-123` nos logs → vê toda a história

## Implementação (bode-api)

### Stack
- **Crate:** `tower-http` com features `request-id`, `propagate-header`

### Setup

```rust
// lib.rs
use axum::http::HeaderName;
use tower_http::request_id::{MakeRequestUuid, PropagateRequestIdLayer, SetRequestIdLayer};

static X_REQUEST_ID: HeaderName = HeaderName::from_static("x-request-id");

pub fn build_router(state: AppState) -> Router {
    routes::create_router()
        .layer(PropagateRequestIdLayer::new(X_REQUEST_ID.clone()))
        .layer(TraceLayer::new_for_http())
        .layer(SetRequestIdLayer::new(X_REQUEST_ID.clone(), MakeRequestUuid))
        .with_state(state)
}
```

### O que cada layer faz

1. **`SetRequestIdLayer`** — verifica se request já tem `x-request-id`. Se não, gera UUID. Se sim, mantém (útil pra tracing cross-service).

2. **`TraceLayer`** — cria tracing span pra cada request. O span captura o request_id.

3. **`PropagateRequestIdLayer`** — adiciona `x-request-id` no response header.

### Ordem dos layers importa
`SetRequestIdLayer` PRIMEIRO (gera), `PropagateRequestIdLayer` ÚLTIMO (escreve no response), `TraceLayer` no meio.

## Uso nos logs

Graças ao tracing span, **todo log dentro do request carrega o request_id automaticamente** — não precisa passar manual:

```rust
async fn me(Authenticated(user): Authenticated) -> Json<AuthUser> {
    tracing::info!(user_id = %user.id, "me requested");
    // JSON output:
    // { "request_id": "abc-123", "user_id": "456", "message": "me requested" }
    Json(user)
}
```

O `request_id` vem do span injetado pelo middleware.

## Cross-service propagation

Se bode-api chama outro serviço, propague o ID:

```rust
// Chamada pro capra-analytics-api
let response = reqwest::Client::new()
    .get("https://other-service/api/thing")
    .header("x-request-id", current_request_id())  // propaga
    .send()
    .await?;
```

Assim, o outro serviço loga com o MESMO ID → rastreabilidade end-to-end.

## Debugging com request_id

### Cenário: user reporta erro

1. Cliente recebe `x-request-id: abc-123` no response
2. Usuário manda pra suporte: "deu erro, id abc-123"
3. Suporte queries nos logs: `request_id="abc-123"`
4. Vê toda a sequência: middleware → auth → DB query → handler → response
5. Identifica ponto exato da falha

Sem request_id: teria que buscar por timestamp/path, arriscado misturar requests.

## Relação com outros conceitos

- `[[Structured Logging]]` — request_id só é útil com logs estruturados (JSON). Em texto puro perderia.
- `[[ADR-BODE-010 Observability Metrics]]` — métricas agregadas por endpoint, request_id é pra eventos específicos

## Referências

- `[[bode-api]]` — projeto
- `[[Structured Logging]]` — complementar
- [tower-http docs — request_id](https://docs.rs/tower-http/latest/tower_http/request_id/)
