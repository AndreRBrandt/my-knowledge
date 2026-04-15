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
| D1 | **Separate Supabase project for staging** | "Near-prod, isolated, real" only meaningful with isolated DB. Schema/data parity, no risk of staging traffic touching prod tables. Cost: ~$25/mo (Pro) — accept. | ADR-BODE-011 |
| D2 | **Routes mounted explicitly at `/api/v2/...` in Rust router** | No `stripPrefix` middleware. Trade: small code repetition; Gain: Traefik config simpler, route is what handler sees, easier debug. | (no ADR, document in API_REFERENCE.md) |
| D3 | **GitHub Environments + required reviewer for prod** | Tag `v*` triggers workflow but waits on manual approval in GH UI. Defends against accidental tag, gives audit trail. Homolog stays auto. | ADR-BODE-012 |
| D4 | **Container image signing (Cosign)** | Sign at build, verify before deploy. Aligns with SLSA Level 2+. Defends against registry compromise. | ADR-BODE-013 |
| D5 | **Automated release with `release-please`** | Reads Conventional Commits → opens PR with semver bump + CHANGELOG. Merge that PR = tag = deploy. Removes manual versioning toil. | ADR-BODE-014 |
| D6 | **Blue-green via Traefik labels** (mesma label, drain por unhealthy) | Modernizes ADR-BODE-004's strategy: instead of script orchestrating two routes, both colors share labels → Traefik load-balances → forcing old color unhealthy drains traffic naturally. Simpler, more robust. | ADR-BODE-015 (refines 004) |
| D7 | **Observability MVP from day 1** | ADR-BODE-010 deferred metrics. Blue-green without golden signals = "deploy and pray". Start with: uptime monitoring (free tier) + `/metrics` endpoint + basic Grafana. Defer full Prometheus stack on VPS until justified. | ADR-BODE-016 (replaces 010) |
| D8 | **Deploy directory `/home/bode-api/`** | Matches existing pattern `/home/bode-analytics/`. Compose + .env.homolog + .env.prod live there. | (convention, no ADR) |

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
                        ├──► C. Compose Fixes ──► D. Homolog Pipeline ──► E. Prod Pipeline (Blue-Green)
B. Router /api/v2 ──────┘                                                       │
                                                                                │
F. Cosign signing ──────────────────────────────────────► (gated into D + E workflows)
G. release-please ──────────────────────────────────────► (orthogonal, parallel)
H. Observability MVP ───────────────────────────────────► (orthogonal, parallel)
```

A and B are independent prerequisites. C depends on both. D depends on A+B+C. E depends on D shipped + GH Environments configured. F/G/H parallelize but should be ready before E ships to truly say we're "modern".

### Recommended sequencing
1. **Sprint 1:** A + B (parallel) → C → unblocks deploy
2. **Sprint 2:** D (homolog working) + start G+H (parallel, low blast radius)
3. **Sprint 3:** E (blue-green prod) + F (Cosign) — couple them so prod ships signed
4. **Sprint 4:** harden H (observability dashboards), retro

---

## Epic A — VPS Provisioning for bode-api homolog
**Goal:** filesystem, secrets, registry, staging DB ready to receive deploys.

### Story A1 — Filesystem & deploy directory
- **A1.1** Create `/home/bode-api/` on VPS (owner `andre:andre`, mode 750)
- **A1.2** Place `docker-compose.homolog.yml` (initially copied from repo manually; later workflow syncs)
- **A1.3** Create `.env.homolog` with staging credentials (manual, never in git)
- **DoD:** `ls -la /home/bode-api/` shows compose file + .env.homolog with correct perms (`.env.homolog` mode 600)

### Story A2 — Staging Supabase project (D1)
- **A2.1** Create new Supabase project `bode-staging` (Pro tier or Free decision — see open question 1)
- **A2.2** Apply schema: dump prod's gold tables structure, recreate in staging
- **A2.3** Seed strategy decision (open question 2): subset of prod data anonymized, fully synthetic, or empty?
- **A2.4** Capture `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_KEY`, JWKS endpoint → into `.env.homolog`
- **DoD:** Rust binary connects to staging DB, `/api/v2/health/db` returns 200 from local pointing to staging URL

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

## Open questions for André

1. **Supabase staging tier:** Free (limited 500MB, 2 projects) or Pro ($25/mo, ~unlimited)? Free likely too small to mirror prod data realistically.
2. **Staging DB seed strategy:** anonymized subset of prod, fully synthetic, or start empty and accept "thin" homolog testing initially?
3. **Alert routing:** email only, Slack webhook (where?), Telegram bot?
4. **release-please scope:** version the whole workspace as one bumped version (recommended for app-style projects), or per-crate (more npm-style)?
5. **Cosign keystore:** GitHub Secrets only (simplest), or KMS (AWS/Hashi Vault — overkill now)?

## What this plan does NOT cover (out of scope for Phase 2)

- RBAC implementation (Epic #8 — separate phase)
- Migration of legacy /api/* routes to /api/v2/* (Strangler Fig — ongoing per-feature)
- Tech debt items in `docs/TECH_DEBT.md` (TD-001 pagination, TD-002 DI traits)

## Acceptance for "Phase 2 complete"

- [ ] Merge to main auto-deploys to homolog within 5min (D)
- [ ] Tag `v*` requires manual approval, then blue-green deploys to prod with zero downtime (E)
- [ ] All deployed images signed and verified (F)
- [ ] Releases versioned and CHANGELOG generated automatically (G)
- [ ] Outage in homolog or prod alerts within 2min (H1)
- [ ] All 6 ADRs (011-016) accepted and merged
