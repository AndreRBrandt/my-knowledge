---
type: decision
status: stable
project: [bode-api]
tags: [adr, deployment, blue-green, zero-downtime]
created: 2026-04-07
related: "[[ADR-BODE-003 Docker Multi-stage Build]], [[Blue-Green Deploy]]"
---

# ADR-BODE-004 Blue-Green Deployment (Zero Downtime)

**Status:** Accepted
**Date:** 2026-04-07

## Context
The platform serves restaurant operations that depend on real-time data. Any downtime during deployment directly impacts business operations. We need a deployment strategy that ensures zero downtime.

## Decision
Implement blue-green deployment using Docker containers behind Traefik reverse proxy:
1. Deploy new version to "green" container alongside running "blue"
2. Health check passes on green
3. Traefik switches traffic to green
4. Blue container is stopped after drain period

## Consequences

### Positive
- Zero downtime during deployments
- Instant rollback by switching back to blue
- Health checks validate the new version before receiving traffic
- Simple mental model for the team

### Negative
- Requires running two containers briefly during switchover
- Slightly more complex deployment scripts
- Need sufficient VPS resources for dual containers

### Risks
- Database migrations must be backwards-compatible (both versions may run simultaneously)

## Alternatives Considered

| Alternative | Why rejected |
|-------------|--------------|
| Rolling restart (single container) | Brief downtime during restart |
| Kubernetes | Over-engineered for single-VPS deployment |
| Serverless (Cloudflare Workers) | Cold starts, limited runtime, harder debugging |

## References
- `[[bode-api]]` — project hub
- `[[Blue-Green Deploy]]` — pattern details
