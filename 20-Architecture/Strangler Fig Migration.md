---
type: architecture
status: living
project: [bode-api, bode-analytics-api]
tags: [migration, strategy, traefik]
created: 2026-04-15
related: "[[bode-api]], [[ADR-BODE-001 Rust Backend Language]]"
---

# Strangler Fig Migration

> Padrão de migração incremental: novo sistema convive com o legado, captura rotas uma a uma até estrangular o antigo. Inspirado no Strangler Fig — figueira que cresce em volta de uma árvore, consumindo-a.

## Contexto no Capra

Migração do backend **TypeScript (Hono)** → **Rust (Axum)** sem big-bang.

```
Antes (só TS):              Durante (strangler):            Depois (só Rust):
─────────────               ──────────────────              ─────────────────
                                                            
   Traefik                    Traefik                         Traefik
      │                          │                                │
      ↓                     ┌────┴────┐                           ↓
bode-analytics-api          ↓         ↓                       bode-api
 (TS, /api/*)       /api/*  /api/v2/*                         (Rust, /*)
                    TS       Rust
                    (legado) (novo)
```

## Regras de convivência

### Roteamento via Traefik
- **`/api/v2/*`** → `bode-api` (Rust), priority 30
- **`/api/*`** → `bode-analytics-api` (TS), priority 20
- **`/*`** → frontend (Vue), priority 10

Priority maior ganha o match. `/api/v2/...` nunca cai no TS.

### Feature migration
Novas features SEMPRE no Rust em `/api/v2/`. Features existentes migram em ordem de prioridade:
1. Features quebradas (reescritas corretamente em Rust)
2. Features que precisam de melhoria significativa
3. Features estáveis (migradas por último)

### Database
Ambos apontam pro mesmo banco (Supabase hoje, self-hosted futuro). Lê/escreve nas mesmas tabelas.

### Frontend
Hoje chama `/api/*` (TS). Ao migrar uma feature, frontend muda chamadas pra `/api/v2/*`. Transparente via proxy.

## Como decidir o que migrar

**Migre agora se:**
- Feature está quebrada ou buggy
- Feature precisa de refatoração significativa
- Feature tem gargalo de performance
- Feature requer feature nova que seria melhor em Rust

**Não migre se:**
- Feature está estável e o custo de migração > valor
- Não há contexto de como refazer no Rust

## Quando acabar

O TS backend pode ser desligado quando:
- Nenhuma rota ativa em `/api/*` está no TS
- Testes garantem que `/api/v2/*` cobre todos os casos
- Frontend não chama mais `/api/*` (só `/api/v2/*`)

Último passo: Traefik manda todo `/api/*` pro Rust, remove rota do TS, desliga container TS.

## Vantagens

- **Zero downtime** — ambos rodam o tempo todo
- **Rollback fácil** — se algo quebra em Rust, swap no Traefik volta pro TS
- **Aprendizado gradual** — equipe aprende Rust uma feature por vez
- **Sem big bang** — risco distribuído no tempo

## Desvantagens

- **Duplicação temporária** — dois backends mantidos simultaneamente
- **Schema compartilhado** — migrações no DB afetam ambos
- **Complexidade de deploy** — dois containers, duas CI/CD

## Referências

- Pattern original: [Martin Fowler — StranglerFigApplication](https://martinfowler.com/bliki/StranglerFigApplication.html)
- `[[bode-api]]` — backend Rust (novo)
- `[[bode-analytics-api]]` — backend TS (legado)
- `[[ADR-BODE-001 Rust Backend Language]]` — motivação da migração
