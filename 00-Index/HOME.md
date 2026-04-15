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

- bode-api Fase 2 — pipeline de deploy (homolog + prod blue-green)

## 🎯 Quick Access
_Links frequentes._

- `[[bode-api]]` — backend Rust em construção
- `[[capra-dbt]]` — pipeline de dados
- `[[dbt Architecture]]` — arquitetura do pipeline

## 📂 Estrutura
```
00-Index/          → MOCs, regras, entry points
10-Domain/         → Negócio (cross-project)
20-Architecture/   → Técnico (cross-project)
30-Decisions/      → ADRs
40-Projects/       → Hubs finos por projeto
50-Sessions/       → Logs temporais
60-References/     → Aprendizados externos
70-Templates/      → Templates de nota
80-Scripts/        → Scripts registrados (SQL, ops, diagnostics)
_attachments/      → Imagens, diagramas
```

## 🆕 Criando nota nova
Use templates de `70-Templates/`:
- `[[Concept]]` — conceito do domínio ou técnico
- `[[Decision]]` — decisão arquitetural (ADR)
- `[[Process]]` — procedimento ou fluxo
- `[[Project Hub]]` — novo projeto
- `[[Session]]` — log de sessão
- `[[Script]]` — script executável
