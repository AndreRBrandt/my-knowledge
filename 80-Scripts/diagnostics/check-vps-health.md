---
type: script
status: stable
project: [infra]
tags: [vps, monitoring, health, docker]
created: 2026-04-15
category: diagnostic
parameters: []
related: ""
---

# check-vps-health

> Verifica saúde geral da VPS: CPU, memória, disco, containers Docker, SSL certs.

## When to use
- Verificação periódica (pode ser cron de 5min)
- Troubleshoot quando algo está lento
- Antes de deploy — confirmar que VPS tem recursos
- Depois de incidente — validar que voltou ao normal

**Read-only.** Diagnóstico puro.

## Pre-requisites
- SSH config com alias `bode-vps` apontando pra `andre@157.230.12.108`
- Chave SSH configurada (`~/.ssh/id_ed25519` ou equivalente)

## Command

```bash
ssh bode-vps "
  echo '--- Uptime & Load ---'
  uptime
  echo
  echo '--- Memória ---'
  free -h
  echo
  echo '--- Disco ---'
  df -h /
  echo
  echo '--- Containers ---'
  docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Image}}'
  echo
  echo '--- Unhealthy containers ---'
  docker ps --filter health=unhealthy --format '{{.Names}}'
  echo
  echo '--- SSL cert expiry (Traefik acme.json) ---'
  docker exec bode-traefik sh -c 'cat /certs/acme.json' 2>/dev/null | head -1
"
```

## Parameters

Nenhum.

## Expected output

### Success (tudo OK)
```
--- Uptime & Load ---
 12:34:56 up 15 days,  4:23, ... load average: 0.45, 0.55, 0.60
--- Memória ---
              total        used        free
Mem:           7.8Gi       1.2Gi       500Mi
--- Disco ---
Filesystem  Size  Used Avail Use%
/dev/vda1    78G   22G   56G  29%
--- Containers ---
NAME                STATUS                      IMAGE
bode-api            Up 2 days (healthy)         ghcr.io/...
bode-traefik        Up 15 days (healthy)        traefik:v3.4
...
--- Unhealthy containers ---
(vazio)
```

### Failure modes
- **Load > 2.0:** CPU sobrecarregada, investigar qual container
- **Mem used > 90%:** memória alta, check `docker stats` pra identificar container
- **Disco > 85%:** risco de encher, rodar `docker system prune`
- **Container unhealthy:** `docker logs <container>` pra investigar
- **SSL < 14 dias:** Traefik deve renovar automaticamente, mas monitorar

## How to invoke

```bash
# Local
./scripts/check-vps-health.sh

# Ou direto
ssh bode-vps "free -h && df -h && docker ps"
```

## Source location

- Script automatizado: `capra-workspace/infra/scripts/resource-monitor.sh` (cron de 5min na VPS)
- Alerta: Telegram via `TG_BOT_TOKEN` + `TG_CHAT_ID` quando thresholds excedidos

## Notes

- A VPS tem 8GB RAM, 78GB SSD
- Thresholds do auto-monitor: disk >85%, RAM >90%, swap >500MB
- Log rotation: Docker local driver max 10MB/file, 3 files por container

## Related

- _(a migrar)_ VPS infrastructure doc
- _(a migrar)_ Monitoring strategy
