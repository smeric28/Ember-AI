# Ember AI

Secure, multi-tenant AI platform for Fireball Industries — LLM gateway, RAG pipelines, and agentic workflows on K3s.

## Stack

| Component | Image | Purpose |
|---|---|---|
| **LiteLLM** | `ghcr.io/berriai/litellm` | OpenAI-compatible API gateway with model routing |
| **n8n** | `docker.n8n.io/n8nio/n8n` | Workflow orchestrator for RAG and automation |
| **pgvector** | `ankane/pgvector` | PostgreSQL + vector embeddings for RAG |
| **Valkey** | `valkey/valkey` | Redis-compatible cache and queue backend |
| **OpenZiti** | `openziti/ziti-edge-tunnel` | Zero-trust overlay network (EmberNet Flux) |
| **ZeroTier** | `zerotier/zerotier` | Admin interface access (sidecar) |

## Models (DeepInfra)

| Name | Model | Role |
|---|---|---|
| `nemotron-120b` | Nemotron 3 Super 120B | Main reasoning engine |
| `nemotron-4b` | Nemotron 3 Nano 4B | Routing, classification, fast tasks |

## Quick Start

```powershell
.\deploy.ps1
wsl -d Ubuntu k3s kubectl get pods -n ember-ai
```

## Documentation

- [Developer Onboarding](docs/onboarding.md) — Setup, first API call, troubleshooting
- [Developer Guide](docs/developer-guide.md) — Full architecture and component reference
- [Architecture](docs/architecture.md) — Design principles
- [Infrastructure](docs/infrastructure.md) — K3s prerequisites and setup
- [Integration](docs/integration.md) — Data flow between components

## Secrets

All secrets injected via GitHub Actions from GitHub Secrets at deploy time.

## License

Proprietary — Fireball Industries LLC
