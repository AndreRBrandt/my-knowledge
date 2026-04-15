---
type: concept
status: stable
tags: [auth, teknisa, zeedhi, http, oracle]
created: 2026-04-15
related: "[[teknisa-crawler]], [[Data Extraction Engine]], [[bode-api]]"
---

# Teknisa BI Auth Flow

> Fluxo de autenticação para a plataforma **Teknisa BI** (Zeedhi backend). Conceito independente da implementação — vale tanto pro crawler Python atual quanto pra port Rust futura.

## Por que este conceito existe canonicamente

A autenticação é o **primeiro requisito** pra qualquer integração com Teknisa. O padrão Zeedhi é específico (não é OAuth padrão, não é JWT padrão) e tem nuances (PHPSESSID + token + headers OAuth-style cohabitando). Documentar aqui evita reverso-engenharia repetida.

## Sequência de autenticação

```
┌─────────────┐                              ┌──────────────────┐
│   Client    │                              │   Teknisa BI     │
└──────┬──────┘                              └────────┬─────────┘
       │                                              │
       │  1. GET /auth/login                          │
       │ ────────────────────────────────────────────►│
       │                                              │
       │  HTML + Set-Cookie: PHPSESSID=<sess_id>      │
       │ ◄────────────────────────────────────────────│
       │                                              │
       │  2. POST /backend_login/index.php/login      │
       │     Body: { EMAIL, PASSWORD, ... }           │
       │     Cookie: PHPSESSID=<sess_id>              │
       │ ────────────────────────────────────────────►│
       │                                              │
       │  { TOKEN, USER, LOGGED, USER_ID,             │
       │    USER_NAME, USER_EMAIL, ... }              │
       │ ◄────────────────────────────────────────────│
       │                                              │
       │  3. Subsequent requests:                     │
       │     Headers: OAuth-Token, OAuth-Hash,        │
       │              OAuth-Project                   │
       │     Cookie: PHPSESSID=<sess_id>              │
       │ ────────────────────────────────────────────►│
```

## Headers críticos pós-login

| Header | Origem | Vida útil |
|---|---|---|
| `OAuth-Token` | Vem da resposta do login (`TOKEN`) | TTL definido pelo servidor (~horas) |
| `OAuth-Hash` | Hash da org, **fixo por plataforma** | Não muda — vem do .env |
| `OAuth-Project` | Product ID, **fixo por plataforma** | Não muda — vem do .env |
| `Cookie: PHPSESSID` | Vem do `Set-Cookie` da página de login | Vinculado à sessão HTTP |

**Importante:** os 4 são necessários. Faltar qualquer um → 401 ou comportamento inconsistente.

## Diferença BI vs Retail

Ambos usam Zeedhi mas com versões diferentes:

| Aspecto | Teknisa BI | Teknisa Retail |
|---|---|---|
| Versão Zeedhi | Novo (Vue SPA, NPM) | v4.23 legacy (Bower) |
| Login payload format | JSON flat: `{ EMAIL, PASSWORD, ... }` | FilterData: `{ requestType, filter[] }` |
| Login response | `{ TOKEN, USER, LOGGED, ... }` direto | `{ dataset: { userData: { TOKEN, ... } } }` aninhado |
| Endpoint principal | QueryRunner (Oracle SQL custom) | 22 backend URLs (relatórios prontos) |

Implementação tem que tratar os dois formatos — o crawler Python tem branch por plataforma em `_parse_login_response`.

## Cache de sessão (recomendado)

Sem cache: cada query nova faz login novo (lento, gera carga, risco de rate limit).

**Padrão:**
- Após login, persiste `{token, phpsessid, oauth_hash, oauth_project, base_url, logged_at}` em arquivo (Python) ou DB/Redis (Rust)
- Antes de cada query: lê cache, checa idade vs TTL → se válido, reutiliza; senão, re-loga
- TTL definido por config (default razoável: 4-8h)
- Em caso de 401 numa query (token expirou no servidor antes do TTL local): re-loga e retenta UMA vez

## Risco de segurança

- **PHPSESSID + TOKEN** no cache local = qualquer um com acesso ao filesystem pode usar sua sessão
- **Crawler Python** mantém em `session_bi.json` (gitignored, mas no FS plain)
- **Port Rust** deve considerar: arquivo encriptado? OS keychain? Variável de processo só (sem persist)? — decisão operacional do Epic K2

## Implementações conhecidas

| Implementação | Stack | Local | Status |
|---|---|---|---|
| Python crawler | requests + dotenv | `bi_projects/projects_script/teknisa_crawler/src/auth.py` | Ativo, validado |
| Rust port | reqwest + serde | `bode-api/crates/bode-sync/src/auth/zeedhi.rs` | Planejado (Phase 2 K2) |
