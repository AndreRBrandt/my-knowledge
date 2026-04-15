---
type: moc
status: living
created: 2026-04-15
---

# Projects MOC

> Índice de todos os projetos. Cada projeto tem um **hub fino** em `40-Projects/` com apenas links curados.

## Active

### Data pipeline
- `[[capra-dbt]]` — dbt project (raw → staging → intermediate → marts)

### Backend
- _(a migrar)_ bode-api — Rust backend (em construção, substitui TS)
- _(a migrar)_ bode-analytics-api — TS backend (legado, sendo substituído)

### Frontend
- _(a migrar)_ capra-ui — framework Vue 3 pra dashboards
- _(a migrar)_ capra-analytics — dashboard app (usa capra-ui)

### Ingestion
- _(a migrar)_ teknisa-crawler — extração API Teknisa
- _(a migrar)_ ifood-crawler — extração iFood
- _(a migrar)_ SugestCard — crawler específico

### Outros
- _(a migrar)_ cardapio-bode — cardápio interativo IA (BodinhaSugere)
- _(a migrar)_ wiki-grupo-no — wiki Docmost + MCP experiment

## Archived / Legacy

- `bi_projects/_legacy_reference/rh_pontomais_gold_layer/` — estrutura BIMachine (descontinuada), mantida como referência pra refatoração

---

## Sobre Project Hubs

Hubs são **finos** — só links curados. Conteúdo vive em `10-Domain/`, `20-Architecture/`, `30-Decisions/`.

Ver `[[Operating Principles]]` → seção "Project Hubs são FINOS".

Template: `[[Project Hub]]`
