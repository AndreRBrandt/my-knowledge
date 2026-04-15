---
type: session
status: stable
date: 2026-04-15
tags: [session, vault, migration, capra-ui]
created: 2026-04-15
---

# 2026-04-15 — Vault Migration Wave 2 (capra-ui + GADRs + Domain)

## Contexto
Continuação direta da sessão anterior (`[[2026-04-15 Vault Setup + bode-api Migration]]`). Pendência de S235: capra-ui (PRIORIDADE ALTA), GADRs globais, domain consolidado. Plano: 4 agentes em background — bloqueados por permissão. Executei manualmente em sequência.

## O que foi feito

### capra-ui ADRs (19)
Em `30-Decisions/capra-ui/`:
- ADR-UI-001 a ADR-UI-019 — distiladas a partir de `capra-ui/docs/adr/*.md`
- Inclui cross-links para architecture concepts (mesmo nome) e GADRs
- Status preservado do source (Accepted/Proposed/Living)

### capra-ui Architecture Concepts (14)
Em `20-Architecture/`:
- Adapter Pattern (UI Layer), Action Bus, Schema Builder, Measure Engine, Filter Registry
- Theme System, Domain Containers, Dimension Discovery
- Component Categories (UI vs Containers vs Analytics), Interaction System
- Core vs App Dependency Boundaries, Generic Composables Layer, Data Loading Patterns
- Framework-First Visual Ownership

### Workspace GADRs (8)
Em `30-Decisions/global/`:
- GADR-001 Strangler Fig TS to Rust Migration
- GADR-002 Trunk-Based Development
- GADR-003 Conventional Commits
- GADR-004 Spec-Driven Workflow with Consultant LLM
- GADR-005 Framework-First UI Development
- GADR-006 VPS as Stepping Stone Before Cloud
- GADR-007 No SQL in Client
- GADR-008 BIMachine Deprecation

### Domain notes (10)
- `10-Domain/organization/` — `[[Filial]]`, `[[Empresa]]`
- `10-Domain/rbac/` — `[[Roles]]`, `[[Permissions]]`, `[[User Permissions Override]]`, `[[RBAC Cascade Model]]`
- `10-Domain/glossary/` — `[[Glossary (consolidated)]]`
- `10-Domain/business-rules/` — `[[Faturamento Calculation]]`, `[[Comparison Layers (3-tier)]]`, `[[Trend Inversion]]`

### Hub novo
- `40-Projects/capra-ui.md`

### MOCs atualizados
- Decisions MOC — incluiu 19 ADR-UI + 8 GADRs
- Architecture MOC — incluiu 14 novos concepts UI + 2 cross-cutting
- Domain MOC — incluiu organization + RBAC + business-rules + glossary
- Projects MOC — adicionou capra-ui hub
- HOME — atualizado estado da migração e quick access

## Decisões

- **Componentes/specs/COMPONENTS.md NÃO foram migrados** — são canônicos no repo capra-ui (Rule 6 do CLAUDE.md). Vault contém apenas decisões e padrões reutilizáveis.
- **Glossary consolidado em 1 nota** — termos com 1-linha não justificam nota dedicada. Termos com nota dedicada (Filial, Roles, etc.) são linkados.
- **Faturamento Calculation virou nota — mantida** — regra crítica de negócio (não é implementação), aplicável a múltiplos projetos.
- **GADR-008 BIMachine Deprecation criada** — formaliza decisão estratégica que estava espalhada em comentários e specs.
- **Empresa.md criada apesar de não ter coluna ainda** — documenta o conceito + o débito explícito de schema.

## Pendente

- Hubs dos projetos secundários: bode-analytics-api, capra-analytics, crawlers (teknisa/ifood/SugestCard), cardapio-bode, wiki-grupo-no
- ADRs específicas do capra-dbt (se houver decisões formais a documentar)
- 60-References/ — vazio; ainda sem aprendizados externos catalogados

## Próximo

Foco volta para **bode-api Fase 2** (pipeline de deploy: graceful shutdown, deploy-homolog.yml, smoke tests, blue-green, rollback, deploy-prod.yml). Issues #35-#42 no GitHub Projects bodedono.

## Context needed (próxima sessão)
- `[[bode-api]]` hub
- Issues #35-#42 abertas
- `[[Blue-Green Deploy]]` concept
- `bode-api/docs/TECH_DEBT.md`
