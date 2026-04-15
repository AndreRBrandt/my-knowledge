---
type: decision
status: stable
scope: workspace
tags: [decision, global, workspace, migration, rust]
created: 2026-04-07
related: "[[Strangler Fig Migration]], [[ADR-BODE-001 Rust Backend Language]], [[bode-api]]"
---

# GADR-001 Strangler Fig Migration: TS Backend → Rust

**Status:** Accepted (ongoing)
**Date:** 2026-04-07

## Context
Backend `bode-analytics-api` (TS/Hono/Node) tem limitações: alto uso de memória, ausência de safety compile-time para pipelines de dados, erros de runtime que vazam pra produção apesar do TypeScript. Rewrite total = risco alto + perda de feature parity. Migrar incrementalmente preserva entregas.

## Decision
Reescrever o backend em Rust (`bode-api`) usando padrão Strangler Fig: novo backend coexiste com o legado, Traefik roteia `/api/v2/*` → Rust e `/api/*` → TS. Features novas nascem em Rust; features quebradas são reescritas em Rust; features estáveis migram incrementalmente. Fim do ciclo: TS decomissionado.

## Consequences

### Positive
- Risco baixo (legado segue funcionando)
- Feature parity preservada durante migração
- Permite aprender Rust em produção sem big-bang
- Cada migração entrega valor incremental

### Negative
- Coexistência prolongada de dois stacks
- Overhead operacional duplicado (deploys, monitoring)
- Tentação de "deixar pra depois" features que precisariam migrar

### Risks
- Migração pode estagnar (parar no meio com 2 backends pra sempre) — mitigado por roadmap explícito de decommission

## Alternatives Considered

| Alternativa | Por quê rejeitada |
|-------------|-------------------|
| Big-bang rewrite | Risco inaceitável, perda de feature parity |
| Manter TS | Não resolve problemas de safety/memória |
| Migrar para Go | Type system mais fraco, sum types ausentes |

## References
- `[[Strangler Fig Migration]]` — concept arquitetural
- `[[ADR-BODE-001 Rust Backend Language]]` — decisão da linguagem (Rust vs alternativas)
- `[[bode-api]]` — projeto novo
