---
type: decision
status: stable
scope: workspace
tags: [decision, global, workspace, infrastructure, vps]
created: 2026-03-27
related: "[[GADR-007 No SQL in Client]], [[bode-api]]"
---

# GADR-006 VPS as Stepping Stone Before Cloud

**Status:** Accepted
**Date:** 2026-03-27

## Context
Stack inicial: Cloudflare Workers + KV (~$35/mês), Vercel, Supabase. Limitações sentidas: CPU 50ms (free) / 30s (paid), KV eventual consistency (~60ms cache), custos fragmentados, sem flexibilidade pra wiki/serviços extras. Necessidade de aprender ops antes de escalar para cloud gerenciada.

## Decision
**Migrar para VPS DigitalOcean ($24/mês fixo) como stepping stone.** Roda todos os containers (API, web, Redis, Traefik, wiki, runner, Postgres dev) em 1 servidor. Operacional manual ensina os trade-offs antes de adotar K8s/cloud.

Quando volume/equipe crescer, migrar para cloud gerenciada com conhecimento operacional adquirido.

## Consequences

### Positive
- Custo previsível (sem surpresa de billing)
- CPU ilimitado (queries OLAP complexas sem worry)
- Redis real (<1ms, LFU, 256MB)
- Visibilidade operacional total (htop, docker stats, logs)
- Aprendizado de ops antes de escalar

### Negative
- Responsabilidade total por security, updates, monitoring
- Single point of failure (1 servidor)
- Sem auto-scaling, sem zero-downtime nativo (~2s downtime no deploy)

### Revisitar quando
- Volume >10k req/min sustentado (auto-scaling necessário)
- Equipe >10 devs (ops overhead justifica managed PaaS)
- Multi-região obrigatório

## Alternatives Considered

| Alternativa | Por quê rejeitada |
|-------------|-------------------|
| Cloudflare Workers + KV | CPU limit, latência KV, custos fragmentados |
| Hetzner ($5/mês) | Sem datacenter LATAM (180ms vs 40ms) |
| AWS EC2 | Custo variável (egress), complexidade |
| Railway/Fly.io | Lock-in, custo variável |

## References
- `[[bode-api]]` — deploy target
- `D01`, `D02` em `wiki-content/12-decisoes-arquiteturais.md` (detalhes operacionais)
- `[[infra-strategy-goals]]` (memory) — intenção estratégica
