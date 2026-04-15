---
type: meta
status: stable
created: 2026-04-15
---

# Operating Principles

> Regras fundamentais de uso deste vault. Este é o documento canônico — em caso de dúvida, consulte aqui.

## Filosofia

Este vault foi desenhado para **context engineering agentic**, não para RAG tradicional.

O objetivo: criar um sistema onde LLMs (Claude) e humanos navegam conhecimento curado, hierarquicamente estruturado, com contexto composto dinamicamente por tarefa — em vez de busca por similaridade em embeddings opacos.

## Princípios fundamentais

### 1. Atomicidade
Uma nota = um conceito. Se uma nota começa a cobrir múltiplos conceitos distintos, quebre em notas menores.

**Por quê:** notas atômicas são **carregáveis individualmente**. LLM pega só o que precisa para a tarefa — sem ruído.

### 2. Link > Duplicar
NUNCA copie conteúdo entre notas. Sempre `[[linke]]`. Se o mesmo fato aparece em 2 lugares, ou ele é **uma nota só** (e as outras linkam) ou **virou uma nota nova**.

**Por quê:** single source of truth. Mudou o fato, muda num lugar só. Links atualizam automaticamente.

### 3. Domain > Project
Organize conhecimento pelo que ele **é**, não pelo onde é usado.

- ✅ `20-Architecture/Auth Flow JWKS.md` (linkado por múltiplos projetos)
- ❌ `bode-api/auth.md` + `capra-ui/auth.md` (duplica)

**Por quê:** mesmo conceito usado em N projetos deve existir uma vez. Projetos linkam.

### 4. Cascata (entrada) + Grafo (conteúdo)
Topo = **cascata curada** (MOCs e Hubs). Navegação top-down previsível.
Base = **grafo de conceitos atômicos**. Links cruzados contextuais.

Profundidade máxima: 3-4 hops. Mais que isso = grafo mal estruturado, refatore.

### 5. Fresh context per task
A cada nova tarefa, o contexto é **recompilado do zero** a partir do vault. Não carrega conhecimento da tarefa anterior "por garantia".

**Por quê:** economia de tokens + alta densidade de sinal por token = melhor qualidade de resposta.

### 6. Token economy
Toda nota carregada no contexto deve **justificar seus tokens**. Se metade do conteúdo é irrelevante pra tarefa, quebre a nota em duas.

### 7. Curadoria > busca
MOCs e Hubs são **curados por humano**. Não confie em auto-geração.

**Por quê:** curadoria encode relevância que similaridade não captura.

## Regras de escrita

### Frontmatter obrigatório

Toda nota começa com frontmatter YAML:

```yaml
---
type: concept | decision | process | reference | session | hub | meta | moc
status: draft | living | stable | archived
project: [bode-api]      # multi-valor, opcional
tags: [auth, security]
created: YYYY-MM-DD
related: "[[Other Note]], [[Another Note]]"
---
```

Campos:
- `type`: identifica template/tipo (ver `70-Templates/`)
- `status`: lifecycle da nota
  - `draft`: rascunho, incompleto
  - `living`: em uso, pode mudar
  - `stable`: consolidado, mudança rara
  - `archived`: desatualizado, mantido por histórico
- `project`: a quais projetos a nota se aplica (se aplicável)
- `tags`: tags transversais (podem cruzar hierarquia)
- `created`: data de criação
- `related`: links para notas relacionadas

### Naming convention

| Tipo | Padrão | Exemplo |
|------|--------|---------|
| Conceito | `Title Case Sem Data.md` | `Blue-Green Deploy.md` |
| Session | `YYYY-MM-DD short title.md` | `2026-04-14 Fase 1 Deploy.md` |
| ADR | `ADR-NNN Title.md` | `ADR-004 Blue-Green Deploy.md` |
| Project Hub | `{project-name}.md` (exato) | `bode-api.md` |
| MOC | `{Theme} MOC.md` | `Domain MOC.md` |

### Estrutura de pastas

```
my-knowledge/
├── 00-Index/          # MOCs, HOME, regras — ponto de entrada
├── 10-Domain/         # Conhecimento de negócio (cross-project)
├── 20-Architecture/   # Conhecimento técnico (cross-project)
├── 30-Decisions/      # ADRs unificados
├── 40-Projects/       # Hubs finos de projetos (apenas links curados)
├── 50-Sessions/       # Logs temporais de trabalho
├── 60-References/     # Aprendizados externos (livros, artigos)
├── 70-Templates/      # Templates por tipo de nota
└── _attachments/      # Imagens, diagramas, PDFs
```

### Project Hubs são FINOS

Hub de projeto **não contém conteúdo** — só **links curados** para o que importa daquele projeto:

```markdown
# bode-api

## Domain
- [[RBAC Model]]
- [[Filial]]

## Architecture
- [[Auth Flow JWKS]]
- [[Strangler Fig Migration]]

## Decisions
- [[ADR-001 Rust Backend]]
- [[ADR-004 Blue-Green Deploy]]

## Active work
- [[50-Sessions/2026-04-14 Fase 1 Deploy]]
```

Se o hub tem parágrafos de conteúdo, esse conteúdo **pertence a outra nota**.

## Anti-patterns (não faça)

- ❌ Notas grandes "por completude"
- ❌ Duplicar conteúdo entre projetos
- ❌ Pasta por projeto pra conceitos compartilhados
- ❌ MOC auto-gerado por script (MOCs são curados)
- ❌ Conhecimento estável misturado com ephemeral na mesma nota
- ❌ Links sem frontmatter
- ❌ Parágrafos de conteúdo em Project Hubs
- ❌ Navegação "flat" sem MOCs (força busca, não navegação)

## Como o Claude usa este vault

O Claude opera via filesystem — lê arquivos diretamente do clone local:

1. **Entry point:** `00-Index/HOME.md`
2. **Navegação top-down:** HOME → MOC apropriado → Hub/Conceito
3. **Exploração lateral:** seguir `[[links]]` relacionados à tarefa
4. **Critério de parada:** contexto suficiente pra tarefa (não ler tudo)

Para isso, o Claude precisa:
- Repo clonado localmente (`C:\Users\ANDRE-BI\my-knowledge`)
- `git pull` antes de operar
- `git add + commit + push` ao editar/criar notas

## Status lifecycle

```
draft → living → stable → archived
 ↑        ↓
 └──── pode voltar se precisar revisão
```

- **draft:** rascunho inicial, pode não ter toda info
- **living:** em uso ativo, evolui conforme aprendizado
- **stable:** consolidado, muda raramente
- **archived:** desatualizado ou substituído, mantido por histórico

## Revisão periódica

Trimestralmente (ou quando sentir que está desorganizado):

1. **Órfãs:** notas sem links entrando nem saindo — decidir se linkam ou arquivam
2. **Drafts velhos:** rascunhos com > 30 dias — completar ou excluir
3. **Stable desatualizado:** notas stable que não refletem mais a realidade — voltar pra living
4. **Hubs:** revisar links curados, tirar o obsoleto

## Evolução desta regra

Este documento também é uma nota — pode evoluir. Mas mudanças são **decisões arquiteturais**:

1. Propor mudança (nova nota em `30-Decisions/ADR-VAULT-NNN`)
2. Avaliar impacto
3. Se aprovado, atualizar este documento
4. Comunicar (sessão no `50-Sessions/`)

---

**Este documento é a regra zero. Todas as outras notas seguem daqui.**
