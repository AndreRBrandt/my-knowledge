---
type: meta
status: stable
created: 2026-04-15
---

# 80-Scripts

> Registry de **scripts nomeados** que buscam dados externos, executam operações e geram relatórios. O vault não duplica dados — guarda **pointers executáveis** pra dados frescos.

## Filosofia

Em vez de:
```
Vault com: "Filial 0001 é Boa Viagem, aberta em 2015..."  (dados estáticos, desatualizam)
```

Usamos:
```
Vault com: Script "list-filiais" + query SQL + descrição de output
           ↓
           Executar pra obter dados FRESCOS sob demanda
```

**Vantagens:**
- Dados nunca desatualizam (vêm do source)
- Vault fica pequeno (metadados, não conteúdo)
- LLM executa script → contexto fresco por tarefa
- Rastreabilidade (script é reproduzível)

## Estrutura

```
80-Scripts/
├── data-queries/      # SQL queries nomeadas (consulta)
├── operations/        # Ops: deploy, restart, migrations
├── diagnostics/       # Troubleshoot: health, logs, metrics
└── reports/           # Geração de relatórios periódicos
```

## Categorias

### data-queries
SQL queries contra Postgres (Supabase). Consulta de estado atual.
Exemplo: "list active filiais", "vendas by domain", "pending user permissions".

### operations
Scripts que EXECUTAM ações (mutação).
Exemplo: `dbt-refresh`, `deploy-homolog`, `db-migration-apply`.

### diagnostics
Investigação / troubleshooting. Read-only.
Exemplo: "check container health", "find slow queries", "memory usage per container".

### reports
Scripts que geram saídas formatadas pra relatórios.
Exemplo: "daily sales report", "monthly ops summary".

## Como usar

### Como humano
1. Abre a nota do script
2. Lê pré-requisitos + parâmetros
3. Executa o comando no terminal
4. Consulta "Expected output" pra validar

### Como Claude
1. Recebe pedido ("me mostra filiais ativas")
2. Busca script relevante em 80-Scripts/
3. Lê parâmetros e pré-requisitos
4. Executa (ou pede pra você executar se requer credencial)
5. Retorna resultado interpretado

## Regras

- **Toda entrada aqui tem frontmatter válido** (ver `[[Script]]` template)
- **Comando é reproduzível** — se depende de env, estado, etc, documentado em Pre-requisites
- **Output esperado está documentado** — facilita validar que rodou certo
- **Scripts que MUTAM precisam warning explícito** no When to use

## Script Template

Use `[[Script]]` em 70-Templates ao criar novo.

## Source of truth

Scripts aqui são **documentação + invocação** de código que vive nos repos:
- `capra-workspace/scripts/` — ops scripts (Node.js)
- `capra-workspace/capra-dbt/` — dbt models
- Projetos individuais — scripts específicos

O vault descreve **como usar**; os repos têm **o código**.
