# Developer Onboarding

Welcome to Ember AI. This guide walks you through getting the platform running locally and making your first API call.

## Prerequisites

| Tool | Version | Install |
|---|---|---|
| WSL 2 + Ubuntu | Latest | `wsl --install -d Ubuntu` |
| K3s (in WSL) | v1.28+ | `curl -sfL https://get.k3s.io \| sh -` |
| PowerShell | 7+ | Pre-installed on Windows |
| `gh` CLI | Latest | `winget install GitHub.cli` |
| `kubectl` (Windows) | Optional | Only needed if not using WSL K3s wrapper |

## 1. Clone the Repo

```bash
git clone https://github.com/Embernet-ai/Ember-AI.git
cd Ember-AI
```

## 2. Understand the Stack

```
┌─────────────────────────────────────────────┐
│ K3s Cluster (WSL Ubuntu)                    │
│                                             │
│  ┌─────────┐  ┌────────┐  ┌──────────────┐ │
│  │ pgvector │  │ Valkey │  │   OpenZiti   │ │
│  │ (RAG DB) │  │(Cache) │  │  (Zero Trust)│ │
│  └────┬─────┘  └───┬────┘  └──────────────┘ │
│       │            │                         │
│  ┌────┴────────────┴────┐                    │
│  │       LiteLLM        │ ← API Gateway      │
│  │  nemotron-120b (main)│                    │
│  │  nemoclaw (agent)    │                    │
│  └──────────┬───────────┘                    │
│             │                                │
│  ┌──────────┴───────────┐                    │
│  │         n8n          │ ← Workflows        │
│  │   RAG ingestion      │                    │
│  │   RAG retrieval      │                    │
│  └──────────────────────┘                    │
└─────────────────────────────────────────────┘
```

## 3. Set Up Secrets

The platform needs API keys injected as K8s secrets. For local dev, create them manually:

```bash
wsl -d Ubuntu k3s kubectl create secret generic ember-ai-env-secrets \
  -n ember-ai \
  --from-literal=POSTGRES_USER=ember \
  --from-literal=POSTGRES_PASSWORD=<your-password> \
  --from-literal=LITELLM_MASTER_KEY=<your-key> \
  --from-literal=DATABASE_URL=postgresql://ember:<password>@pgvector-svc:5432/ember_ai \
  --from-literal=DEEPINFRA_API_KEY=<your-key> \
  --from-literal=NVIDIA_API_KEY=<your-key> \
  --from-literal=OPENAI_API_KEY=<your-key> \
  --from-literal=ANTHROPIC_API_KEY=<your-key> \
  --from-literal=GEMINI_API_KEY=<your-key> \
  --from-literal=KEYCLOAK_CLIENT_ID=<client-id> \
  --from-literal=KEYCLOAK_CLIENT_SECRET=<client-secret> \
  --from-literal=ZITI_IDENTITY_JSON='{}'
```

> **Note**: In production, secrets are managed by the External Secrets Operator pulling from Bitwarden SM. You never need to manually create them outside of local dev.

## 4. Deploy

```powershell
.\deploy.ps1
```

This applies all K8s manifests in dependency order to K3s via WSL.

## 5. Verify

```bash
# Check all pods are running
wsl -d Ubuntu k3s kubectl get pods -n ember-ai

# Check LiteLLM health
wsl -d Ubuntu k3s kubectl port-forward svc/litellm-svc 4000:4000 -n ember-ai &
curl http://localhost:4000/health
```

## 6. Make Your First API Call

```bash
# Chat completion using the main engine (Nemotron 120B)
curl http://localhost:4000/v1/chat/completions \
  -H "Authorization: Bearer <LITELLM_MASTER_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "nemotron-120b",
    "messages": [{"role": "user", "content": "What is Ember AI?"}]
  }'
```

```bash
# Agent call using NemoClaw (with tool support)
curl http://localhost:4000/v1/chat/completions \
  -H "Authorization: Bearer <LITELLM_MASTER_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "nemoclaw",
    "messages": [{"role": "user", "content": "Summarize the latest safety report"}],
    "tools": [{"type": "function", "function": {"name": "search_documents", "parameters": {"type": "object", "properties": {"query": {"type": "string"}}}}}]
  }'
```

## 7. Import RAG Workflows into n8n

1. Port-forward n8n: `wsl -d Ubuntu k3s kubectl port-forward svc/n8n-svc 5678:5678 -n ember-ai`
2. Open `http://localhost:5678`
3. Go to **Settings → Import from File**
4. Import `workflows/rag-ingestion.json` and `workflows/rag-retrieval.json`
5. Update the Postgres credential node with your connection details

## Key Files

| Path | What it is |
|---|---|
| `k8s/` | All Kubernetes manifests (applied in order by `deploy.ps1`) |
| `k8s/08-litellm-config.yaml` | Model routing config — add or change LLM providers here |
| `workflows/` | n8n workflow JSONs for RAG pipelines |
| `docs/developer-guide.md` | Full architecture and component reference |
| `.github/workflows/deploy.yaml` | CI/CD pipeline for automated deployment |

## Troubleshooting

| Issue | Fix |
|---|---|
| Pods stuck in `Pending` | Check PVC: `kubectl get pvc -n ember-ai` |
| LiteLLM `401` errors | Verify `LITELLM_MASTER_KEY` matches the secret |
| ESO `SecretStore` not syncing | Ensure BW access token is created in `external-secrets` namespace |
| OpenZiti tunnel `CrashLoopBackOff` | Check identity JSON is valid and `/dev/net/tun` exists in WSL |

## Getting Help

- Check the [Developer Guide](developer-guide.md) for component details
- Review open issues: `bd ready` (requires Dolt running)
