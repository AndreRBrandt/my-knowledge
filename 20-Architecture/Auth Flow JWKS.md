---
type: architecture
status: living
project: [bode-api]
tags: [auth, jwt, jwks, supabase, security]
created: 2026-04-15
related: "[[bode-api]], [[ADR-BODE-009 Zero Vendor Lock-in]], [[RBAC Model]]"
---

# Auth Flow JWKS

> Fluxo de autenticação do bode-api: valida JWT emitido pelo Supabase usando **JWKS** (chaves assimétricas ES256). Zero shared secret, zero runtime dependency no Supabase após startup.

## Por que JWKS, não HS256

Supabase migrou sua auth de HMAC-SHA256 (shared secret `JWT_SECRET`) pra **ES256** (chave privada no Supabase, pública via JWKS endpoint).

**Antes (HS256):**
- Cliente e servidor compartilham o segredo
- Rotação de segredo quebra todos os tokens em trânsito
- Cliente (bode-api) precisa do segredo pra validar

**Agora (ES256 via JWKS):**
- Supabase assina com chave privada (que ele tem, nós não)
- bode-api valida com chave pública (baixada via JWKS endpoint)
- Rotação: Supabase publica nova chave no JWKS, bode-api recarrega
- Zero segredo compartilhado

## Fluxo

```
1. STARTUP
   bode-api → GET https://{project}.supabase.co/auth/v1/.well-known/jwks.json
   Resposta: { "keys": [ { "kid": "abc", "kty": "EC", "crv": "P-256", ... } ] }
   bode-api: HashMap<kid, DecodingKey> em memória

2. REQUEST
   Cliente → Authorization: Bearer eyJhbGciOiJFUzI1NiIsImtpZCI6ImFiYyJ9.{payload}.{signature}
   
   bode-api:
   a) Decode header → extrai kid="abc"
   b) HashMap lookup → DecodingKey
   c) Valida assinatura ES256 com chave pública
   d) Se OK, extrai claims: { sub, email }
   e) Carrega user do DB (load_auth_user) pelo sub
   f) Retorna Authenticated(AuthUser)

3. HANDLER
   Recebe Authenticated(user) como parâmetro
   User tem: id, email, role, filiais, permissions
```

## Estrutura de código

```rust
// bode-core/auth.rs — domínio puro
pub struct AuthUser {
    pub id: String,
    pub email: String,
    pub role: String,
    pub filiais: Vec<i32>,
    pub permissions: HashMap<String, String>,
}

// bode-server/auth/provider.rs — trait abstrato
pub trait AuthProvider: Send + Sync {
    fn validate(&self, token: &str) -> Result<Claims, AppError>;
}

// bode-server/auth/supabase.rs — implementação concreta
pub struct SupabaseAuthProvider {
    keys: HashMap<String, DecodingKey>,
    audience: String,
}

// bode-server/middleware/auth.rs — extractor
pub struct Authenticated(pub AuthUser);

impl FromRequestParts<AppState> for Authenticated {
    // 1. extract bearer token
    // 2. auth.validate(token) → Claims
    // 3. load_auth_user(pool, claims.sub) → AuthUser
    // 4. return Authenticated(user)
}
```

## Decisões chave

### 1. Trait AuthProvider
Permite trocar Supabase por outro provider sem mudar o middleware. Ver `[[ADR-BODE-009 Zero Vendor Lock-in]]`.

### 2. JWT não carrega role/permissions
JWT tem apenas `sub` (user id) e `email`. Tudo o mais (role, filiais, permissions) vem do DB via `load_auth_user`.

**Por quê:** admin panel pode mudar role/permissions sem invalidar tokens em trânsito.

### 3. Cache de chaves em memória
JWKS é buscado UMA VEZ no startup. Validação offline depois.

**Trade-off:** se Supabase rotacionar a chave, bode-api precisa restart. Em produção, isso é aceitável (blue-green já reinicia).

### 4. Audience fixa "authenticated"
Supabase emite tokens com `aud: authenticated`. Validamos esse claim pra evitar aceitar tokens de outros contextos.

### 5. AppError::Auth centralizado
Todo erro de auth (token inválido, user não encontrado, expirado) vira `AppError::Auth(msg)` → 401. Mensagens detalhadas vão pro log, cliente vê só "unauthorized".

## Endpoints protegidos

```rust
// Rotas públicas
GET /health          // 200 "ok"
GET /health/db       // 200 { status, latency_ms }

// Rotas protegidas (exigem Authenticated extractor)
GET /api/v2/me       // 200 AuthUser JSON | 401
```

## Migração pós-Supabase

Quando migrar pra auth próprio (ver `[[ADR-BODE-009 Zero Vendor Lock-in]]`):
1. Novo `AuthProvider` concreto (`OwnAuthProvider`) com JWT gerado por nós
2. Tokens antigos (Supabase) e novos (próprio) coexistem durante transição via `iss` claim
3. Frontend migra de `supabase.auth.signIn()` pra `POST /api/v2/auth/login`

Middleware não muda — só o provider injetado.

## Referências

- `[[bode-api]]` — projeto
- `[[ADR-BODE-009 Zero Vendor Lock-in]]` — estratégia que motiva o trait
- `[[RBAC Model]]` — modelo de permissões resolvidas pelo `load_auth_user`
- [RFC 7517 — JWKS](https://datatracker.ietf.org/doc/html/rfc7517)
- [RFC 7518 — JWS Algorithms (ES256)](https://datatracker.ietf.org/doc/html/rfc7518)
