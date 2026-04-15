---
type: decision
status: stable
scope: workspace
tags: [decision, global, workspace, security, api]
created: 2026-03-15
related: "[[GADR-006 VPS as Stepping Stone Before Cloud]], [[bode-api]], [[Adapter Pattern (UI Layer)]]"
---

# GADR-007 No SQL in Client (API Mediates All Queries)

**Status:** Accepted
**Date:** 2026-03-15

## Context
Versão inicial: frontend construía SQL e enviava via Supabase RPC (`execute_analytics_sql`). Problemas críticos: lógica exposta no bundle, vetor de SQL injection, credentials no client, impossível auditar/cachear no servidor.

## Decision
**Toda query de dados passa pela API (capra-api Workers ou bode-api Rust).** Frontend envia JSON estruturado (`CapraQuery`); servidor constrói SQL com query builder validado.

```typescript
// PROIBIDO no frontend
const sql = buildSQL(schema, query)
await supabase.rpc('execute_analytics_sql', { query_text: sql })

// CORRETO
await fetch('/api/query', {
  method: 'POST',
  headers: { Authorization: `Bearer ${jwt}` },
  body: JSON.stringify({ schema: 'vendas', query }),
})
```

`supabase-query-builder.ts` vive **exclusivamente** no servidor. NUNCA importado pelo frontend.

## Consequences

### Positive
- Zero SQL injection do client
- Service-role keys nunca chegam ao bundle
- Cache server-side possível (Redis)
- Auditoria centralizada (todas queries logam no servidor)
- Schemas mudáveis sem deploy de frontend

### Negative
- Latência adicional vs RPC direto (mitigada por cache)
- API deve cobrir todos os padrões de query (ou frontend fica preso)

## References
- AP-13 (Anti-pattern: SQL no Cliente) em `docs/context/ANTIPATTERNS.md`
- `[[Adapter Pattern (UI Layer)]]` — mecanismo frontend
- `[[bode-api]]` — futuro mediador único (Strangler Fig)
