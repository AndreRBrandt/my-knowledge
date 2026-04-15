---
type: script
status: draft
project: []
tags: []
created: {{date:YYYY-MM-DD}}
category: data-query | operation | diagnostic | report
parameters: []
related: ""
---

# {{title}}

> Uma frase: o que o script faz e quando usar.

## When to use
Situações que disparam a execução deste script:
- Cenário 1
- Cenário 2

## Pre-requisites
- Credenciais necessárias (ex: `.env.ops` com `DB_HOST`, `DB_USER`)
- Ferramentas (ex: `pnpm`, `psql`, `gh`)
- Acessos (ex: VPN ativa, Tailscale conectado)

## Command / Query

```bash
# comando exato, com placeholders em {}
pnpm ops dbt:refresh --days {N}
```

ou

```sql
-- Query SQL direta (se for data-query)
SELECT coluna
FROM schema.tabela
WHERE condicao = {PARAM};
```

## Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `{N}` | int | 3 | Dias pra refresh |
| `{PARAM}` | string | - | Filtro obrigatório |

## Expected output

### Success
```
Expected: saída esperada em caso de sucesso
```

### Failure modes
- Erro comum 1 — causa + remediação
- Erro comum 2 — causa + remediação

## How to invoke

### Via terminal local
```bash
cd capra-workspace
pnpm ops dbt:refresh
```

### Via Claude (se registrado)
"Roda o script [[Nome do Script]]"

## Dependencies / related

- `[[Related Concept]]` — o que esse script exercita
- `[[Related Project]]` — projeto que possui

## Source location

- Script file: `capra-workspace/scripts/...`
- Linha relevante: `link com line number se aplicável`

## Notes

Contexto adicional, histórico de mudanças, quirks, etc.
