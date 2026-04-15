---
type: moc
status: living
created: 2026-04-15
---

# HOME

> Ponto de entrada do vault. Comece aqui.

## 📖 Regras
- `[[Operating Principles]]` — regras de uso, escrita, navegação

## 🧭 Maps of Content (navegação por tema)
- `[[Projects MOC]]` — todos os projetos (hubs)
- `[[Domain MOC]]` — conhecimento de negócio
- `[[Architecture MOC]]` — conhecimento técnico
- `[[Decisions MOC]]` — ADRs (decisões arquiteturais)

## 🚀 Scripts registry
Scripts nomeados executáveis (SQL queries, ops, diagnostics):
- `[[80-Scripts/README|Scripts overview]]`
- Categorias: `data-queries/`, `operations/`, `diagnostics/`, `reports/`

## 🔴 Active Work
_Sessões recentes e trabalho em andamento._

- bode-api Fase 2 — pipeline de deploy (homolog + prod blue-green) — issues #35-#42
- Vault migration — última leva (capra-ui ADRs+concepts, GADRs, domain) em 2026-04-15. Ver `[[2026-04-15 Vault Migration Wave 2]]`.

## 🎯 Quick Access
_Links frequentes._

- `[[bode-api]]` — backend Rust em construção
- `[[capra-ui]]` — framework Vue
- `[[capra-dbt]]` — pipeline de dados
- `[[Glossary (consolidated)]]` — vocabulário cross-project
- `[[RBAC Cascade Model]]` — modelo mental de permissions

## 📂 Estrutura
```
00-Index/          → MOCs, regras, entry points
10-Domain/         → Negócio (cross-project): organization/, rbac/, glossary/, business-rules/
20-Architecture/   → Técnico (cross-project)
30-Decisions/      → ADRs: bode-api/, capra-ui/, global/
40-Projects/       → Hubs finos por projeto
50-Sessions/       → Logs temporais
60-References/     → Aprendizados externos
70-Templates/      → Templates de nota
80-Scripts/        → Scripts registrados (SQL, ops, diagnostics)
_attachments/      → Imagens, diagramas
```

## 📊 Estado da migração (2026-04-15)
**Migrado:**
- bode-api: 10 ADRs + hub + 8 architecture concepts
- capra-ui: 19 ADRs + hub + 14 architecture concepts
- Global: 8 GADRs (workspace decisions)
- Domain: 2 organization + 4 RBAC + 3 business-rules + glossário consolidado
- dbt: architecture + medallion + hub + 7 dbt-domains
- Scripts: 3 iniciais

**Pendente:**
- Hubs dos outros projetos (bode-analytics-api, capra-analytics, crawlers, cardapio-bode, wiki-grupo-no)

## 🆕 Criando nota nova
Use templates de `70-Templates/`:
- `[[Concept]]` — conceito do domínio ou técnico
- `[[Decision]]` — decisão arquitetural (ADR)
- `[[Process]]` — procedimento ou fluxo
- `[[Project Hub]]` — novo projeto
- `[[Session]]` — log de sessão
- `[[Script]]` — script executável
