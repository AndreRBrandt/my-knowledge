---
type: decision
status: stable
scope: workspace
tags: [decision, global, workspace, methodology, llm]
created: 2026-04-08
related: "[[GADR-005 Framework-First UI Development]]"
---

# GADR-004 Spec-Driven Workflow with Consultant LLM

**Status:** Accepted
**Date:** 2026-04-08

## Context
Único dev BI lidera implementação. LLM (Claude/Gemini) tem capacidade de implementar autonomamente, mas isso cria dois problemas: (1) André não entende o código que vai manter; (2) decisões arquiteturais ficam opacas, surgindo gambiarras "para entregar". Especialmente crítico em Rust (linguagem nova pra equipe).

## Decision
Ciclo: **spec → backlog (GitHub Projects) → implementação guiada por André → wiki**.

- Specs descrevem **O QUE** e **POR QUE**, nunca **COMO**. Sem código.
- Backlog em `bodedono/*` GitHub Projects (não ClickUp).
- André implementa cada PR linha a linha; LLM atua como **consultor** (explica, revisa, sugere alternativas).
- Autonomia da LLM cresce com domínio do código (mais autonomia em capra-ui maduro; consultoria pesada em bode-api Rust).
- Documentação consolidada na wiki ao fim de cada ciclo.

## Consequences

### Positive
- André domina cada decisão e cada linha
- Bus factor não cresce artificialmente com LLM
- Specs documentam decisão antes do código (raciocínio explícito)
- Refactor consciente em vez de "código que LLM cuspiu"

### Negative
- Velocidade menor que LLM autônoma
- Curva de aprendizado em Rust é o gargalo real

## Alternatives Considered

| Alternativa | Por quê rejeitada |
|-------------|-------------------|
| LLM autônoma | André perde domínio do código |
| Pair programming sem spec | Decisões diluídas em chat, sem registro |
| Spec com pseudocódigo | Vaza para implementação prematura |

## References
- `[[capra-platform-strategy]]` — visão de produto
- `[[workflow-spec-driven]]` (memory) — operacional
