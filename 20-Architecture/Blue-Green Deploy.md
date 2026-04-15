---
type: architecture
status: living
project: [bode-api]
tags: [deployment, zero-downtime, traefik]
created: 2026-04-15
related: "[[bode-api]], [[ADR-BODE-004 Blue-Green Deployment]], [[Multi-stage Docker Build]]"
---

# Blue-Green Deploy

> Estratégia de deploy onde duas versões (blue = atual, green = nova) rodam simultaneamente por segundos. Traefik switcha o tráfego quando green passa health check. Rollback = switcha de volta.

## Conceito

```
Antes do deploy:           Durante switch:              Depois:
───────────────            ──────────────                ──────
                                                        
Traefik → blue             Traefik                      Traefik → green
(v1.0.0)                      │ │                       (v1.1.0)
                              ↓ ↓                       
                            blue  green                 (blue stopped)
                            v1.0  v1.1
                            health health
                                   ↓ passed
                            switch traffic
```

## Passos do deploy

### 1. Pre-deploy
- Validar que imagem nova existe no GHCR
- Validar que migrações do DB são backwards-compat (ambas versões rodam simultaneamente)

### 2. Start green
```bash
docker compose -f docker-compose.green.yml up -d api-green
```
- Container `bode-api-v2-green` na porta diferente ou host diferente
- Traefik AINDA aponta pra `bode-api-v2` (blue)

### 3. Health check green
```bash
for i in {1..30}; do
  if curl -sf http://bode-api-v2-green:8000/health; then
    echo "Green healthy"
    break
  fi
  sleep 1
done
```
- Tenta 30s. Se falhar, aborta deploy (green fica, mas tráfego continua em blue).

### 4. Switch traffic
- Atualiza label Traefik: prioridade do green passa a ser maior que blue
- Traefik detecta em ~1s e começa a rotear pro green
- Requests em andamento no blue terminam (drain)

### 5. Drain period (30s)
```bash
sleep 30
```
- Requests long-running terminam
- Blue ainda responde, só não recebe novos

### 6. Stop blue
```bash
docker compose stop api-blue
docker compose rm -f api-blue
```

### 7. Cleanup
- Green vira o novo blue (rename conceptual)
- Próximo deploy: green deploy ao lado do atual

## Rollback

Se algo der errado **depois** do switch:

### Opção A: rollback rápido (blue ainda no ar, pré-stop)
```bash
# Reverter priority do Traefik → blue recebe tráfego novamente
# Tempo: < 10s
```

### Opção B: rollback via pull (blue já parado)
```bash
# Puxar imagem N-1 do GHCR
docker pull ghcr.io/bodedono/bode-api:v1.0.0
# Start como green, health check, switch
# Tempo: < 2min
```

## Requisitos do código

### Graceful shutdown
Binary deve tratar SIGTERM:
```rust
// main.rs
let shutdown = async {
    tokio::signal::ctrl_c().await.ok();
    // ou SIGTERM via signal::unix
};

axum::serve(listener, app)
    .with_graceful_shutdown(shutdown)
    .await?;
```
Sem isso, docker stop espera timeout e manda SIGKILL → requests em flight morrem.

### Stateless
Container não deve depender de estado local entre requests. Se precisa de cache, usa Redis externo.

### Health check meaningful
`/health` deve retornar 200 só se o app realmente pode atender requests. Não só "processo vivo" — `/health/db` testa conexão DB (se crítico).

### Backwards-compat migrations
Durante switch, blue e green rodam juntos. Migration do DB não pode quebrar blue.
- ✅ ADD COLUMN nullable
- ✅ CREATE TABLE nova
- ❌ DROP COLUMN (que blue ainda usa)
- ❌ RENAME COLUMN (que blue ainda usa nome antigo)

Mudanças breaking precisam de 2 deploys:
1. Deploy N: adiciona nova coluna, código escreve em AMBAS
2. Wait period + backfill
3. Deploy N+1: código para de escrever/ler a velha
4. Deploy N+2: DROP COLUMN velha

## Implementação no Capra

- **Traefik** como reverse proxy (já roda na VPS)
- **Docker Compose** pra declarar blue + green
- **Scripts bash** pra orquestrar passos (scripts/deploy-blue-green.sh — a criar)
- **Health check** via Dockerfile `HEALTHCHECK` + endpoint `/health` e `/health/db`
- **Traefik labels** com priority pra switchar

## Alternativas rejeitadas

- **Rolling restart:** brief downtime (1-2s)
- **Kubernetes rolling update:** over-engineered pra single VPS
- **Red-Black:** similar mas mais complexo

## Referências

- `[[bode-api]]` — projeto
- `[[ADR-BODE-004 Blue-Green Deployment]]` — decisão
- [Martin Fowler — Blue-Green Deployment](https://martinfowler.com/bliki/BlueGreenDeployment.html)
