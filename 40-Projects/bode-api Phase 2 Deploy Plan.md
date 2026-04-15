---
type: plan
status: draft
project: [bode-api]
tags: [deploy, ci-cd, infra, phase-2]
created: 2026-04-15
related: "[[bode-api]], [[Blue-Green Deploy]], [[ADR-BODE-004 Blue-Green Deployment]], [[ADR-BODE-006 Three Environments]], [[ADR-BODE-010 Observability Metrics]]"
---

# bode-api — Phase 2 Deploy Pipeline Plan

> **Status:** DRAFT — awaiting André's validation before issue creation.
> Last updated: 2026-04-15 (S237 follow-up).

## Why this plan exists

Phase 1 closed (PR #43, merged 2026-04-14): Dockerfile, compose files, CI gates with 92.5% coverage, JSON logging, request ID. Phase 2 (issues #35-#42) was originally scoped as "deploy pipeline" but two things forced an expansion:

1. **VPS reality check (S237):** the canonical VPS state diverged significantly from memory (14 containers vs 8, Cloudflare Origin Cert vs Let's Encrypt, DNS already provisioned, neighboring legacy stack with /api priority 20). Three real bugs surfaced in the existing `docker-compose.homolog.yml`.
2. **Modern practices research:** GitHub Spec Kit / AWS Kiro / Anthropic SDD patterns plus current blue-green via Traefik labels research (primfeed 2025, straypaper) show six concrete improvements worth adopting from day one — cheaper to bake in now than retrofit.

This plan turns those into a structured Epic → Story → Task → DoD breakdown so we can spec-validate each unit before writing code.

## Decisions to lock in (André confirmed 2026-04-15)

| # | Decision | Rationale | Will become ADR |
|---|---|---|---|
| D1 | **Staging DB = `bode-postgres-dev` self-hosted on VPS** + **Supabase Auth Free project** for JWKS | $0 marginal cost. Forces app to run against vanilla Postgres → validates ADR-009 abstraction → accelerates Eixo 2 da migração (Supabase → self-hosted). Avoids paying $25/mo for infra that will be ripped out. Auth stays on Supabase Free project (projects são gratuitos, só DB Pro é pago). | ADR-BODE-011 |
| D2 | **Routes mounted explicitly at `/api/v2/...` in Rust router** | No `stripPrefix` middleware. Trade: small code repetition; Gain: Traefik config simpler, route is what handler sees, easier debug. | (no ADR, document in API_REFERENCE.md) |
| D3 | **GitHub Environments + required reviewer for prod** | Tag `v*` triggers workflow but waits on manual approval in GH UI. Defends against accidental tag, gives audit trail. Homolog stays auto. | ADR-BODE-012 |
| D4 | **Container image signing (Cosign)** | Sign at build, verify before deploy. Aligns with SLSA Level 2+. Defends against registry compromise. | ADR-BODE-013 |
| D5 | **Automated release with `release-please`** | Reads Conventional Commits → opens PR with semver bump + CHANGELOG. Merge that PR = tag = deploy. Removes manual versioning toil. | ADR-BODE-014 |
| D6 | **Blue-green via Traefik labels** (mesma label, drain por unhealthy) | Modernizes ADR-BODE-004's strategy: instead of script orchestrating two routes, both colors share labels → Traefik load-balances → forcing old color unhealthy drains traffic naturally. Simpler, more robust. | ADR-BODE-015 (refines 004) |
| D7 | **Observability stack from day 1: métricas + logs** | ADR-BODE-010 deferred metrics. Blue-green without golden signals = "deploy and pray". Stack: `/metrics` Prometheus endpoint + uptime monitoring + **Loki+Grafana pra logs em tempo real** (telemetria). | ADR-BODE-016 (replaces 010) |
| D8 | **Deploy directory `/home/bode-api/`** | Matches existing pattern `/home/bode-analytics/`. Compose + .env.homolog + .env.prod live there. | (convention, no ADR) |
| D9 | **Metadata-driven domain architecture** (princípio constitucional) | Toda funcionalidade nova tem entidade de domínio explícita (tabela + repository) + CRUD via API. Filtro de PR: "isso poderia ser configurado via admin sem mexer no código?". Permite produtificação futura como low-code para não-técnicos (Eixo 3 da migração). | ADR-BODE-017 |
| D10 | **Notification adapter pattern (Teams + email hoje)** | Alertas (Epic H1, J) emitidos via trait `Notifier` com adapters concretos (`TeamsAdapter`, `EmailAdapter`). Adicionar Slack/Telegram no futuro = só nova impl. | (parte do ADR-017) |
| D11 | **Staging DB seed via script puxando view Teknisa** | Começa vazio. Script pull de view exposta no Postgres do Teknisa (banco prod) → bode_homolog. Reutiliza acesso já existente, dado real com volume real. PII: tratamento decidido na implementação (TBD). | (parte do ADR-011) |
| D12 | **release-please: workspace versionado como um** | Bode-api é app deployável (não library). 1 versão, 1 tag, 1 CHANGELOG. Per-crate só faria sentido se publicasse em crates.io. | (parte do ADR-014) |
| D13 | **Cosign keystore: GitHub Secrets** | Adequado pro porte (single VPS, time pequeno, app interno). KMS é overkill sem requisito de compliance. | (parte do ADR-013) |

## Real VPS state (verified S237)

See `[[VPS Infrastructure Status]]` (memory) for full inventory. Key facts for this plan:
- 14 containers running including legacy `bode-api-homolog` (Hono, ghcr.io/andrerbrandt/...)
- Traefik v3.4 in `/home/infra/traefik/`, file provider with `security-headers@file` middleware ready
- TLS via Cloudflare Origin Cert (NOT Let's Encrypt) — `cert resolver` label must be removed from our compose
- DNS `bi-homolog.bodedono.com.br` already exists ✓
- Self-hosted runner ON the VPS — no SSH needed, runs as `andre` user
- 99 GB free disk, 6.2 GB available RAM — comfortable for adding our stack + observability

## Bugs in current `docker-compose.homolog.yml` (must fix before deploy)

| Bug | Fix |
|---|---|
| `traefik.http.routers.*.tls.certresolver=letsencrypt` | Remove (VPS uses CF Origin Cert via default store) |
| Missing `healthcheck:` block | Add explicit healthcheck (curl/wget against `/api/v2/health` inside container) |
| Routes `/health` won't match `/api/v2/health` from Traefik | Solved by D2 (explicit /api/v2 mount in Rust router) |

## Epic structure

### Dependency graph
```
A. VPS Provisioning ────┐
                        ├──► C. Compose Fixes ──► D. Homolog Pipeline ──┐
B. Router /api/v2 ──────┘                                               │
                                                                        │
I. Repository abstractions (TD-002) ────────────────────────────────────┤──► E. Prod Pipeline (Blue-Green)
                                                                        │
K. Data Extraction Engine (Teknisa BI) ──► K8 seed_homolog ──┐          │
                                                             │          │
                                                             ▼          │
                               (popula bode_homolog) ────────┴──────────┤
                                                                        │
F. Cosign signing ──────────────────────────────────────► (gated into D + E workflows)
G. release-please ──────────────────────────────────────► (orthogonal, parallel)
H. Observability MVP (metrics) ─────────────────────────► (orthogonal, parallel)
J. Telemetry stack (Loki+Grafana) ──────────────────────► (orthogonal, parallel — engine K9 emite pra ele)
```

A and B are independent prerequisites. C depends on both. D depends on A+B+C. **I (repository abstractions / TD-002) is hard prerequisite for E**. F/G/H/J parallelize. **K (Engine + Teknisa BI) é grande mas paralelo** — só K8 (seed_homolog) bloqueia validação completa de homolog (D ship sem dado real é deploy-only validation).

### Recommended sequencing
1. **Sprint 1:** A1+A2+A3 + B + I + **K1+K2+K3 (engine core + auth + 1º conector)** (paralelo) → C → unblocks deploy
2. **Sprint 2:** D (homolog funcionando) + **K4+K5+K6 (outputs + queries + regression test)** + começa G+H+J (paralelo, baixo blast radius)
3. **Sprint 3:** E (blue-green prod) + F (Cosign) + **K7+K8 (CLI extract + seed_homolog roda em homolog real)**
4. **Sprint 4:** harden H+J + **K9 (logging end-to-end via Loki)** + retro

---

## Epic A — VPS Provisioning for bode-api homolog
**Goal:** filesystem, secrets, registry, staging DB ready to receive deploys.

### Story A1 — Filesystem & deploy directory
- **A1.1** Create `/home/bode-api/` on VPS (owner `andre:andre`, mode 750)
- **A1.2** Place `docker-compose.homolog.yml` (initially copied from repo manually; later workflow syncs)
- **A1.3** Create `.env.homolog` with staging credentials (manual, never in git)
- **DoD:** `ls -la /home/bode-api/` shows compose file + .env.homolog with correct perms (`.env.homolog` mode 600)

### Story A2 — Staging DB on VPS + Supabase Auth Free + Teknisa seed script (D1, D11)
- **A2.1** Inventory `bode-postgres-dev` (postgres:16-alpine, host port 5433, on `public` net?) — confirm reachable from inside docker network and from runner
- **A2.2** Create database `bode_homolog` inside `bode-postgres-dev` (separate from any other dev DBs there); create role `bode_api_homolog` with limited privileges
- **A2.3** Apply schema: dump structure from prod Supabase (`pg_dump --schema-only`), restore into `bode_homolog`
- **A2.4** Create new Supabase project `bode-auth-staging` (Free tier — projects são free, só DB Pro é pago) **only for Auth / JWKS**
- **A2.5** Capture credentials → `.env.homolog`:
  - `DATABASE_URL=postgres://bode_api_homolog:***@bode-postgres-dev:5432/bode_homolog` (internal docker network)
  - `SUPABASE_AUTH_URL=https://<bode-auth-staging>.supabase.co`
  - `SUPABASE_ANON_KEY=...`
  - JWKS endpoint resolved from auth URL
- **A2.6** Bridge networking: ensure `bode-postgres-dev` and `bode-api-v2-homolog` share a docker network (likely add to `backend`)
- **DoD:** Rust binary connects to `bode_homolog` from inside VPS network, `/api/v2/health/db` returns 200; auth login flow works against staging Auth project

### Story A4 — Seed homolog (delegada para Epic K)
**REVISADO:** acesso ao Teknisa NÃO é via Postgres view direto — é via API HTTP autenticada (Zeedhi/QueryRunner) com lógica de auth, session token, headers OAuth. **Já existe** o `[[teknisa-crawler]]` (Python, validado, ~5 queries com sample CSV) que faz isso.

A seed do homolog foi extraída pra **Epic K (Data Extraction Engine)**, story K8 — usa a engine genérica em Rust que substitui (e generaliza) o crawler Python.

A4 propriamente dita só fica como dependência: "K8 concluído → seed funciona em homolog". Sem stories próprias.

### Story A3 — GHCR org packages
- **A3.1** Verify `bodedono` org has Container Registry enabled
- **A3.2** Confirm `GITHUB_TOKEN` of `bodedono/bode-api` has `packages: write` (Settings → Actions → Workflow permissions)
- **A3.3** First manual push `ghcr.io/bodedono/bode-api:bootstrap` to create the package, set visibility (private)
- **A3.4** Verify self-hosted runner can pull (test `docker pull` as `andre` user)
- **DoD:** `docker pull ghcr.io/bodedono/bode-api:bootstrap` succeeds on VPS

---

## Epic B — Router refactor: mount routes at `/api/v2/...` (D2)
**Goal:** Rust handlers see the same path Traefik delivers — no middleware translation.

### Story B1 — Nest router under `/api/v2`
- **B1.1** Refactor `crates/bode-server/src/routes/mod.rs` to nest existing routers under `/api/v2`
- **B1.2** Update unit/integration tests to hit new paths
- **B1.3** Update `docs/API_REFERENCE.md` and any internal docs referencing old paths
- **DoD:** `cargo test` green; `curl localhost:8000/api/v2/health` returns 200; `curl localhost:8000/health` returns 404

---

## Epic C — Compose & infra alignment fixes
**Goal:** `docker-compose.homolog.yml` reflects real VPS Traefik setup.

### Story C1 — Cert + healthcheck alignment
- **C1.1** Remove `traefik.http.routers.bode-api-v2-homolog.tls.certresolver=letsencrypt` label
- **C1.2** Verify `tls=true` alone serves valid cert (default store via `tls.yml`)
- **C1.3** Add explicit `healthcheck:` block (interval 30s, timeout 3s, start_period 15s, retries 3) hitting `http://127.0.0.1:8000/api/v2/health`
- **DoD:** Manual run `docker compose -f docker-compose.homolog.yml config` validates; manual deploy serves valid TLS at `bi-homolog.bodedono.com.br/api/v2/health`

---

## Epic D — Deploy Homolog Pipeline (#35, #37, #42)
**Goal:** Merge to main → image built, signed, pushed, deployed; smoke tests pass; old container drained gracefully.

### Story D1 — `deploy-homolog.yml` workflow (#37)
- **D1.1** Trigger via `workflow_run` after `CI` workflow succeeds on main
- **D1.2** Login to GHCR with `GITHUB_TOKEN`
- **D1.3** Build & push dual tag (`:homolog` rolling + `:homolog-<sha>` immutable)
- **D1.4** Cosign sign the SHA tag (depends on Epic F)
- **D1.5** Sync compose to `/home/bode-api/`, `docker compose pull`, `up -d --remove-orphans`
- **D1.6** Health check loop (30 retries × 2s = 60s max) bypassing CF (`--resolve` to localhost or proper UA header)
- **D1.7** `docker compose stop -t 90` of previous container (graceful drain — relies on shutdown.rs from #41)
- **D1.8** `docker image prune -f --filter "until=24h"` to keep disk in check
- **DoD:** Merge to main → within 5min, new image deployed at `bi-homolog.bodedono.com.br/api/v2/health` returning 200

### Story D2 — Smoke tests pós-deploy (#42)
- **D2.1** Public smoke script: curl `/api/v2/health`, `/api/v2/health/db` with retry
- **D2.2** Authenticated smoke: acquire JWT (Supabase staging), hit one protected endpoint, validate JSON shape
- **D2.3** Integrate into `deploy-homolog.yml` as final step; failure rolls back via re-tagging previous SHA
- **DoD:** Smoke test job in CI shows green for healthy deploy, red + auto-rollback for broken deploy

---

## Epic E — Deploy Prod Pipeline (Blue-Green, #36, #38, #39, #40)
**Goal:** Tag `v*` → human approval → integration tests → blue-green deploy with automated rollback.

### Story E0 — GitHub Environment "production" setup (D3)
- **E0.1** Create GH Environment "production" in `bodedono/bode-api`
- **E0.2** Add required reviewer = André
- **E0.3** Restrict deployment branches to tags matching `v*`
- **DoD:** `gh api repos/bodedono/bode-api/environments/production` returns expected config

### Story E1 — Blue-green deploy script (#39, refined per D6)
- **E1.1** Script accepts SHA, deploys `bode-api-prod-<color>` container with **same Traefik labels** as live (so Traefik balances new + old)
- **E1.2** Wait for new color healthcheck = healthy
- **E1.3** Force old color healthcheck unhealthy (e.g., file-based flag the `/health` handler reads, OR `docker exec` removing the container's healthcheck label) → Traefik drains
- **E1.4** `docker compose stop -t 90` old color
- **DoD:** Script tested on homolog: zero 5xx during switchover (verified by parallel `wrk` or `vegeta` smoke during deploy)

### Story E2 — Rollback script (#40)
- **E2.1** Script accepts target SHA, re-tags `ghcr.io/bodedono/bode-api:prod-<sha>` as `:prod`
- **E2.2** Invokes E1 script to deploy that SHA
- **E2.3** Manual trigger only (`workflow_dispatch` from Actions UI)
- **DoD:** From Actions UI, can pick a previous `prod-<sha>` tag, click run, prod returns to that version in <2min

### Story E3 — `deploy-prod.yml` workflow (#38)
- **E3.1** Trigger on `push: tags: ['v*']`
- **E3.2** Bind to GitHub Environment "production" (gates on E0)
- **E3.3** After approval: run integration tests (suite TBD; minimum: smoke + 1 RBAC happy path)
- **E3.4** Build & push `:prod` + `:prod-<sha>`, Cosign sign
- **E3.5** Invoke E1 blue-green script
- **E3.6** Smoke test (E.D2 reused with prod URL)
- **E3.7** Auto-invoke E2 rollback if E3.6 fails
- **DoD:** Tag `v0.1.0` → approval → tests → blue-green → smoke → end-to-end <10min, zero downtime measured

---

## Epic F — Container image signing (Cosign, D4)
**Goal:** every deployed image is provably built by our CI.

### Story F1 — Cosign keypair + signing
- **F1.1** Generate cosign keypair offline; store private key + password as GitHub Secrets (`COSIGN_PRIVATE_KEY`, `COSIGN_PASSWORD`)
- **F1.2** Public key committed to repo at `cosign.pub`
- **F1.3** After build & push in `deploy-homolog.yml` and `deploy-prod.yml`: `cosign sign --key env://COSIGN_PRIVATE_KEY ghcr.io/bodedono/bode-api@sha256:...`
- **DoD:** `cosign verify --key cosign.pub ghcr.io/bodedono/bode-api:homolog-<sha>` returns valid signature

### Story F2 — Verify before deploy
- **F2.1** In deploy step (D1.5 / E1), run `cosign verify` before `docker compose up`
- **F2.2** Workflow fails hard if signature invalid
- **DoD:** Manually pushing an unsigned image to GHCR results in failed deploy with "signature verification failed"

---

## Epic G — Automated release with `release-please` (D5)
**Goal:** Conventional Commits → automatic semver bump + CHANGELOG + tag.

### Story G1 — release-please workflow
- **G1.1** Add `.github/workflows/release-please.yml` (runs on push to main)
- **G1.2** `release-please-config.json` configured for Rust + workspace versioning
- **G1.3** First run opens PR titled `chore(main): release X.Y.Z` with bumped Cargo.toml + appended CHANGELOG entries
- **G1.4** Merging that PR creates tag `vX.Y.Z` → triggers `deploy-prod.yml`
- **DoD:** After merging two `feat:` commits + one `fix:`, release-please opens a release PR with correct minor bump and properly grouped CHANGELOG

---

## Epic H — Observability MVP (D7, replaces deferred ADR-010)
**Goal:** detect deploys-gone-bad fast; have evidence for debugging.

### Story H1 — Uptime monitoring
- **H1.1** Sign up for healthchecks.io (free tier) OR BetterStack
- **H1.2** Configure HTTP check on `https://bi.bodedono.com.br/api/v2/health` (1min interval) and `bi-homolog`
- **H1.3** Alert routing: email + (Telegram bot? Slack webhook? — open question 3)
- **DoD:** Killing prod container manually triggers alert within 2min

### Story H2 — `/metrics` endpoint
- **H2.1** Add `axum-prometheus` (or equivalent) to `bode-server`
- **H2.2** Expose `/metrics` on internal route only (not via Traefik public)
- **H2.3** Standard metrics: `http_requests_total`, `http_request_duration_seconds` (histogram), `http_requests_in_flight`
- **DoD:** `curl localhost:8000/metrics` returns valid Prometheus exposition

### Story H3 — Prometheus + Grafana on VPS (deferred decision)
- **H3.1** docker-compose for `prom/prometheus` + `grafana/grafana` in `/home/infra/observability/`
- **H3.2** Prometheus scrapes `bode-api:8000/metrics` over docker network
- **H3.3** Grafana dashboard with golden signals (RED method)
- **DoD:** Grafana shows live request rate, p50/p95/p99 latency, error rate per route
- **NOTE:** Defer until H1+H2 prove insufficient; uptime + raw metrics may be enough for v1.

---

## Epic I — Repository abstractions (promotes TD-002, prerequisite for Epic E)
**Goal:** trait-based repository injection so swapping Supabase Auth for self-hosted auth (Eixo 2 da migração) is a config/wiring change, not a rewrite.

**Source:** `bode-api/docs/TECH_DEBT.md` TD-002 — promoted into Phase 2 because:
- Prod (Epic E) shipping with concrete `SupabaseAuthProvider` baked into handlers locks us in
- Cost of doing the trait extraction NOW: ~1 day
- Cost of doing it AFTER prod is on Supabase: re-deploy + risk of behavior change

### Story I1 — Extract `UserRepository` trait in `bode-core`
- **I1.1** Define `pub trait UserRepository: Send + Sync` with method(s) currently provided by `load_auth_user`
- **I1.2** Move trait definition to `bode-core/src/repositories/user.rs` (new module)
- **I1.3** Keep `bode-core` zero-I/O — trait only, no impl
- **DoD:** `cargo check -p bode-core` green; trait is `dyn`-compatible (object-safe)

### Story I2 — Implement trait in `bode-db`
- **I2.1** Move current `load_auth_user` body into `PgUserRepository` struct in `bode-db/src/repositories/user.rs`
- **I2.2** `impl UserRepository for PgUserRepository`
- **I2.3** Constructor takes `PgPool`
- **DoD:** Existing tests pass against new `PgUserRepository`

### Story I3 — Wire via `Arc<dyn UserRepository>` in `AppState`
- **I3.1** `AppState` field changes from concrete to `Arc<dyn UserRepository>`
- **I3.2** `main.rs` constructs `Arc::new(PgUserRepository::new(pool))` and injects
- **I3.3** Auth middleware reads from state via trait
- **DoD:** All routes work end-to-end via trait dispatch; integration test green

### Story I4 — Update `TECH_DEBT.md` and ADR-009
- **I4.1** Move TD-002 from "Active" to "Resolved" in `TECH_DEBT.md`
- **I4.2** Add validation note in ADR-BODE-009 (zero vendor lock-in) — abstraction now proven

---

## Epic J — Telemetria: logs em tempo real (Loki + Grafana, D7)
**Goal:** ver logs estruturados da API em tempo real + buscar histórico + alertar em padrões.

### Story J1 — Loki + Grafana stack
- **J1.1** `docker-compose.yml` em `/home/infra/observability/` com `grafana/loki` + `grafana/grafana` + `grafana/promtail`
- **J1.2** Promtail coleta logs do `bode-api-v2-homolog` e `bode-api-v2-prod` via docker socket-proxy
- **J1.3** Loki indexa por labels (`container`, `environment`, `level`)
- **J1.4** Grafana datasource pra Loki, dashboard inicial: tail-f, latência por rota (parseando log), errors recentes
- **J1.5** Traefik route privada `grafana.bodedono.com.br` (não público — atrás de auth básica ou IP whitelist)
- **DoD:** abrir `grafana.bodedono.com.br` mostra logs do bode-api em tempo real, query LogQL `{environment="prod", level="error"}` retorna últimos erros

### Story J2 — Alertas via adapter (D10)
- **J2.1** Trait `Notifier` em `bode-core`: `async fn send(&self, alert: Alert) -> Result<()>`
- **J2.2** Impls em `bode-server`: `TeamsAdapter` (webhook do Teams), `EmailAdapter` (SMTP — configurar provider, ex: SendGrid free tier ou Resend)
- **J2.3** Grafana Alerting → webhook → endpoint `/internal/alert` no bode-api → roteia pros notifiers configurados
- **J2.4** Configuração de qual adapter pra qual alerta vive em DB (metadata-driven, D9): tabela `alert_routes (severity, channel, target)`
- **DoD:** criar alerta no Grafana ("error rate > 5% por 5min") → dispara → chega no Teams + email; trocar adapter = só editar tabela

### Story J3 — Retention + storage
- **J3.1** Loki configurado com retention 30d (homolog) / 90d (prod)
- **J3.2** Storage: filesystem local (volume mount) — sem object storage por enquanto (decidir migração quando passar de N GB)
- **DoD:** logs antigos são compactados/removidos automaticamente conforme política

---

## Epic K — Data Extraction Engine v1 (Teknisa BI) — `bode-sync`
**Goal:** engine de extração genérica e configurável em Rust, com Teknisa BI como primeiro conector. Substitui (e generaliza) o `[[teknisa-crawler]]` Python.

**Identidade do crate `bode-sync`:** "engine de extração + outras tools internas". Não vira repo separado por enquanto, mas é projetado standalone (binários CLI rodáveis fora do contexto bode-api).

**Source:** discussão S237 (2026-04-15). Princípios:
- Connector trait + Query como entidade de primeira classe (D9 / ADR-017)
- CLI standalone — usável por outros projetos (cron, terminal, futuramente admin builder)
- Sample CSVs do Python crawler viram **regression fixtures** dos testes Rust (critério: "port retorna o mesmo dado da versão Python")
- Logging estruturado por run (tracing spans, correlatable nos dashboards do Epic J)

### Story K1 — Core traits em `bode-core` + impls básicas em `bode-sync`
- **K1.1** Trait `Connector` em `bode-core/src/extraction/connector.rs`: `async fn authenticate`, `async fn run_query(query: &Query) -> Result<Rows>`, `fn name() -> &'static str`
- **K1.2** Struct `Query` em `bode-core/src/extraction/query.rs`: `id`, `connector_name`, `payload` (enum: SQL, ApiCall, etc.), `output_schema`
- **K1.3** Struct `Rows` (resultado tipado): vetor de mapas `column_name → Value`
- **K1.4** Trait `Output` em `bode-core/src/extraction/output.rs`: `async fn write(rows: Rows) -> Result<usize>` (retorna # rows escritas)
- **K1.5** Struct `Extractor` em `bode-sync/src/extractor.rs`: orquestra `Connector + Query → Output`, emite tracing spans
- **DoD:** `cargo check -p bode-core` e `-p bode-sync` verde; traits `dyn`-compatible

### Story K2 — Auth Zeedhi (port `auth.py`)
- **K2.1** Módulo `bode-sync/src/auth/zeedhi.rs`: login (`POST /backend_login/index.php/login`), parse response, build headers (`OAuth-Token`, `OAuth-Hash`, `OAuth-Project`, `Cookie: PHPSESSID`)
- **K2.2** Cache de sessão TTL (`session_bi.json` formato compatível com Python — facilita rollback) — config TTL via env
- **K2.3** Re-login automático quando 401/403 ou expired
- **K2.4** Credenciais via env vars (`TEKNISA_BI_USER`, `TEKNISA_BI_PASS`, `TEKNISA_BI_URL`) — **NUNCA hardcoded**
- **DoD:** `bode_sync::auth::zeedhi::get_session()` retorna sessão válida; cache hit não chama servidor novamente até expirar

### Story K3 — Connector Teknisa BI (port `query_runner.py`)
- **K3.1** Struct `TeknisaBiConnector` em `bode-sync/src/connectors/teknisa_bi.rs`
- **K3.2** `impl Connector for TeknisaBiConnector`
- **K3.3** `run_query` monta payload (`POST /backend/index.php/runQuery`), parse `dataset.queryResult`, mapa pra `Rows`
- **K3.4** Tratamento de erro: HTTP failures, JSON parse errors, queries com erro Oracle → `CoreError` apropriado
- **K3.5** Timeout configurável (default 120s) + retry com backoff exponencial em erros transientes
- **DoD:** `connector.run_query(simple_query)` retorna rows reais do Teknisa BI

### Story K4 — Output adapters (Postgres + stdout JSON/CSV)
- **K4.1** `PostgresOutput` em `bode-sync/src/outputs/postgres.rs`: recebe `target_table` + `upsert_on` (colunas pra `ON CONFLICT`), faz `INSERT ... ON CONFLICT DO UPDATE`
- **K4.2** `JsonOutput` em `bode-sync/src/outputs/json.rs`: serializa rows pra stdout ou arquivo (ad-hoc exploration)
- **K4.3** `CsvOutput` em `bode-sync/src/outputs/csv.rs`: idem com CSV
- **K4.4** Output selecionado em runtime via flag CLI ou config
- **DoD:** mesma query roteada pra cada output produz resultado correto; Postgres upsert idempotente

### Story K5 — Migrar 5 queries Python pra `bode-sync/queries/teknisa/vendas/`
- **K5.1** Copiar `q1_resumo_filial.sql`, `q2_top_produtos.sql`, `q3_turno.sql`, `q4_ticket_medio.sql`, `q5_detalhe_itens.sql` do crawler Python pra `bode-sync/queries/teknisa/vendas/`
- **K5.2** Para cada query, copiar `amostra.csv` como `<query>.fixture.csv` em `bode-sync/tests/fixtures/teknisa/vendas/`
- **K5.3** Adicionar metadata YAML por query: `<query>.meta.yaml` com `{id, connector, output_schema, target_table?, upsert_on?}`
- **DoD:** estrutura no lugar; `bode-sync` carrega queries via `Query::from_file(path)`

### Story K6 — Regression test contra fixtures Python
- **K6.1** Test integration `bode-sync/tests/teknisa_bi_regression.rs` — para cada query em `queries/teknisa/`, executa via Rust, compara com fixture CSV
- **K6.2** Comparação row-equality (não byte) — tolera ordering, formatação numérica diferente
- **K6.3** Skipped por default em CI (precisa credenciais reais); roda manualmente via `cargo test --features teknisa-bi-live`
- **K6.4** Modo "snapshot update": flag pra regenerar fixtures se Teknisa mudou dado
- **DoD:** `cargo test --features teknisa-bi-live` passa pras 5 queries com dados de hoje

### Story K7 — CLI genérica `extract`
- **K7.1** Binary `bode-sync/src/bin/extract.rs`
- **K7.2** Args: `--connector teknisa-bi --query vendas/q1_resumo_filial --output postgres|json|csv [--target-table X] [--params key=val,...]`
- **K7.3** Substituição de parâmetros nas queries (ex: `:data_inicio` → CLI flag)
- **K7.4** Logging humano-readable em terminal, JSON em scripts (flag `--log-format`)
- **K7.5** Exit codes apropriados (0 sucesso, 1 erro de query, 2 erro de auth, etc.)
- **DoD:** `cargo run --bin extract -- --connector teknisa-bi --query vendas/q1_resumo_filial --output json` imprime JSON com dados

### Story K8 — CLI `seed_homolog` (substitui Story A4 original)
- **K8.1** Binary `bode-sync/src/bin/seed_homolog.rs` em cima do `extract`
- **K8.2** Lê config `bode-sync/config/seed_homolog.yaml`: lista de `{query: 'vendas/q1', target_table: 'gold_vendas', upsert_on: 'id'}`
- **K8.3** Para cada entry: roda extract → escreve no `bode_homolog`
- **K8.4** Modo `--dry-run` (mostra o que faria sem escrever)
- **K8.5** Logs estruturados por query: rows extraídas, rows escritas, duração, erros
- **DoD:** `cargo run --bin seed_homolog -- --env homolog` popula `bode_homolog` com dados reais do Teknisa; idempotente (rodar 2x não duplica)

### Story K9 — Logging estruturado + observabilidade
- **K9.1** Tracing span por execução: `extraction.run` com fields `connector`, `query_id`, `output`, `rows`, `duration_ms`, `status`
- **K9.2** Erros emitem context completo (query SQL truncada, HTTP status, payload — sem credenciais)
- **K9.3** Logs em formato compatível com Loki (Epic J): JSON com labels indexados
- **K9.4** Run history opcional persistida em DB (tabela `extraction_runs` — alimenta dashboards de monitoring no futuro)
- **DoD:** rodar `seed_homolog` e ver os spans completos no Grafana/Loki; todas queries têm span individual

---

## Open questions — RESOLVED (2026-04-15)

Todas as decisões locked. Operacionais TBD ficam pra implementação:

| Pergunta | Decisão |
|---|---|
| Seed staging | Script de pull de view Teknisa → bode_homolog (D11). Detalhes (qual view, PII handling) decididos quando atacar Epic A4 |
| Alert routing | Teams + email com adapter pattern (D10). Slack/Telegram = nova impl no futuro |
| release-please scope | Workspace versionado como um (D12) |
| Cosign keystore | GitHub Secrets (D13) |

## TBDs operacionais (decididos quando atacar a story)

- **A4.1** Qual view específica do Teknisa? (André sabe)
- **A4.2** PII handling — filtra na origem, scrub no destino, ou aceita real
- **J1.5** Auth do Grafana — basic auth, IP whitelist, ou Supabase Auth integrado
- **J2.2** SMTP provider — SendGrid free, Resend, ou SMTP do Teams

## What this plan does NOT cover (out of scope for Phase 2)

- RBAC implementation (Epic #8 — separate phase)
- Migration of legacy /api/* routes to /api/v2/* (Strangler Fig — ongoing per-feature)
- Outros tech debt items em `docs/TECH_DEBT.md` (TD-001 paginação, TD-003 tarpaulin) — TD-002 PROMOVIDO pra Epic I deste plano
- **Frontend / capra-ui improvements** — workstream separado, ver `[[capra-ui Improvements Plan]]` (placeholder)
- **Self-hosted DB + auth migration end-to-end (Eixo 2 completo)** — fase futura. Phase 2 só ENCAIXA AS FUNDAÇÕES (Epic A2 staging em Postgres VPS, Epic I trait abstractions)
- **Admin builder / low-code UI (Eixo 3)** — fase futura. Phase 2 prepara a fundação metadata-driven (D9, ADR-017) que vai permitir construir o builder em cima depois sem rewrite.

## Acceptance for "Phase 2 complete"

- [ ] Merge to main auto-deploys to homolog within 5min (D)
- [ ] Tag `v*` requires manual approval, then blue-green deploys to prod with zero downtime (E)
- [ ] All deployed images signed and verified (F)
- [ ] Releases versioned and CHANGELOG generated automatically (G)
- [ ] Outage in homolog or prod alerts within 2min (H1) via Teams + email
- [ ] Logs em tempo real visíveis em Grafana (J)
- [ ] Repository abstractions resolved (I, TD-002 → Resolved)
- [ ] **Data Extraction Engine v1 funcional: CLI `extract` roda queries do Teknisa BI em modo ad-hoc; `seed_homolog` popula `bode_homolog` com dados reais; regression tests verde contra fixtures Python (K)**
- [ ] Toda nova feature passa no filtro "isso poderia ser CRUDado via admin?" (D9 / ADR-017 enforcement)
- [ ] All 7 ADRs (011-017) accepted and merged
