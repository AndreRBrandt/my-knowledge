---
type: session
status: living
project: [my-knowledge, bode-api]
tags: [vault-setup, migration, context-engineering]
created: 2026-04-15
related: "[[HOME]], [[Operating Principles]], [[bode-api]], [[dbt Architecture]]"
---

# 2026-04-15 Vault Setup + bode-api Migration

> Criação do vault `my-knowledge`, definição de filosofia (context engineering vs RAG), limpeza de projetos legado, migração completa de conhecimento do bode-api + dbt.

## Done

### Vault infra
- Repo privado criado: `github.com/AndreRBrandt/my-knowledge`
- Clone local: `C:\Users\ANDRE-BI\my-knowledge`
- SSH config multi-conta configurado:
  - `~/.ssh/id_ed25519` → `brandt-andre` (trabalho, bodedono)
  - `~/.ssh/id_ed25519_personal` → `AndreRBrandt` (pessoal)
  - Alias `github-personal` no SSH config pra repos pessoais

### Vault estrutura
- `00-Index/` — HOME, Operating Principles (regras canônicas), 4 MOCs
- `10-Domain/` — conhecimento de negócio (7 domínios dbt criados)
- `20-Architecture/` — padrões técnicos (10 docs criadas)
- `30-Decisions/` — ADRs (10 ADRs bode-api migradas)
- `40-Projects/` — hubs finos (bode-api + capra-dbt criados)
- `50-Sessions/` — esta nota
- `60-References/` — vazio (a popular)
- `70-Templates/` — 6 templates (Concept, Decision, Process, Hub, Session, Script)
- `80-Scripts/` — NOVO: registry de scripts executáveis (SQL, ops, diagnostics)
- `_attachments/` — mídia

### Filosofia documentada
- `[[Operating Principles]]` — regras canônicas do vault
- Context engineering > RAG (notas atômicas navegáveis, não embeddings)
- Cascata (MOCs) + Grafo (conceitos) como navegação híbrida
- Scripts registrados como alternativa a duplicar estado externo no vault

### Limpeza filesystem
Deletados (git history preservado nos remotes):
- `bi_projects/capra_ui/` (standalone, legado)
- `bi_projects/capra-analytics/` (orphaned, sem git remote)
- `bi_projects/teknisa_banco_temp/` (duplicata, migrou pra dbt)
- `bi_projects/tknisa_silver_layer/` (exploração, migrou pra dbt)
- `bi_projects/hikvision-ifood/` (abandonado Jan/26)
- `bi_projects/_deploy*/` (configs Vercel vazias)
- `bi_projects/projects/` (vazia)
- `capra-workspace/.claude/worktrees/*` (3 worktrees Fev/26)

Arquivado:
- `bi_projects/rh_pontomais_gold_layer/` → `bi_projects/_legacy_reference/` (estrutura BIMachine, referência pra refatoração)

### Migração bode-api (completa)
- 10 ADRs → `[[30-Decisions/bode-api]]/` (BODE-001 a BODE-010)
- Project hub: `[[bode-api]]`
- 8 architecture concepts novos:
  - `[[Strangler Fig Migration]]`
  - `[[Auth Flow JWKS]]`
  - `[[Crate Boundaries]]`
  - `[[Multi-stage Docker Build]]`
  - `[[Blue-Green Deploy]]`
  - `[[Structured Logging]]`
  - `[[Request ID Propagation]]`
  - `[[Two-pipeline Testing Strategy]]`

### Migração dbt (completa)
- `[[dbt Architecture]]` — canonical (NÃO ARQUITETURA_DBT.md do repo — aquele é legacy)
- `[[Medallion Architecture]]` — padrão conceitual
- `[[capra-dbt]]` — project hub
- 7 domain notes em `10-Domain/dbt-domains/` (Core, Vendas, Financeiro, CMV, Avaliacoes, RH, Estoque)

### Scripts registry
- `[[80-Scripts/README]]` — filosofia e uso
- `[[Script]]` template em 70-Templates/
- 3 scripts iniciais:
  - `[[dbt-refresh]]` (operation)
  - `[[list-active-filiais]]` (data-query)
  - `[[check-vps-health]]` (diagnostic)

## Decisions (sessão)

- **Tool:** Obsidian + Git plugin (free for personal, burla Obsidian Sync pago via GitHub)
- **Alternative:** VS Code + Foam (100% OSS, mesmo conteúdo). Arquivos portáveis entre tools.
- **Sync:** GitHub private repo (começa local, adiciona server se necessário depois)
- **VPS pessoal:** deferred (não precisa pra começar)
- **Vault unificado:** 1 vault `my-knowledge` (separar em work/personal/studies depois se necessário)
- **BIMachine oficialmente obsoleto** — removido de docs canônicas
- **Realidade atual confirmada:** Supabase (agora) → self-hosted Postgres (futuro), via bode-api Rust
- **dbt é canônico** pro pipeline de dados (raw → staging → intermediate → marts)

## Pending (próxima sessão)

### O que falta migrar pro vault

**Projetos sem hub ainda:**
- capra-ui (15 ADRs ativas, components, composables) — PRIORIDADE ALTA
- bode-analytics-api (TS legado, hub fino "sendo substituído")
- capra-analytics (dashboard Vue)
- capra-workspace (monorepo meta-hub)
- Crawlers: teknisa_crawler, ifood-crawler, SugestCard
- cardapio-bode (BodinhaSugere IA)
- wiki-grupo-no (MCP experiment)

**Architecture:**
- Hub Pattern (GADR-001)
- Adapter Pattern (capra-ui)
- Action Bus (capra-ui)
- Component Categories (capra-ui)

**Decisions:**
- 7 GADRs globais (de `bi_projects/docs/contexto/DECISOES_GLOBAIS.md`)
- capra-ui ADRs (pular 005, 007, 009, 017 que são obsoletos)

**Domain (a migrar de `bi_projects/docs/integracao/GLOSSARIO_GLOBAL.md` e tknisa docs):**
- Filiais (9 com códigos)
- Unidades (BDN, Burguer, Italiano)
- RBAC Model (AuthUser, Role, Permission) — de bode-api
- Glossário consolidado

## Next

Quando retomar:

1. **Verificar** se André instalou Obsidian + Git plugin (Community Plugins → "Obsidian Git")
2. **Escolher próximo bloco** de migração: **capra-ui** (maior volume e relevância) é minha recomendação
3. **Continuar Fase 2 bode-api** (deploy pipeline) — issues #35-#42 no GitHub projects

### Passo zero da próxima sessão
```bash
cd C:/Users/ANDRE-BI/my-knowledge && git pull
```
Isso garante que eu (Claude) tenho o vault atualizado antes de operar.

## Context needed (próxima sessão)

**Leituras essenciais (na ordem):**
1. `[[HOME]]` — entry point
2. `[[Operating Principles]]` — regras de uso
3. Esta nota — o que foi feito

**Depois, conforme o trabalho:**
- Migrar capra-ui → ler `capra-workspace/capra-ui/docs/adr/INDEX.md` como ponto de partida
- Continuar bode-api Fase 2 → ler `[[bode-api]]` hub + issues #35-42

**Estado atual do bode-api (Fase 1 concluída):**
- PR #43 merged 2026-04-14
- Fase 2 pendente: graceful shutdown (#41), deploy-homolog.yml (#37), blue-green script (#39), smoke tests (#42), etc.

**Onde NÃO procurar:**
- `bi_projects/docs/ARQUITETURA_DBT.md` — versão antiga, menciona BIMachine. Usar `[[dbt Architecture]]` no vault.
- `bi_projects/_legacy_reference/` — referência histórica, não usar como fonte canônica.
