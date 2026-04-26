# Sagewai Python API Client

Official Python API client for the [Sagewai](https://sagewai.ai) agent infrastructure platform.

Sagewai's official client libraries exist to make integration seamless for developers in every language. They provide typed interfaces, proper authentication (API key, JWT, OAuth), error handling, and streaming support -- so you can build on Sagewai without needing to work directly with HTTP endpoints.


## Install

```
pip install sagewai-client
```

Requires Python 3.10+.


## Quick Start

```python
from sagewai_client import SagewaiClient

client = SagewaiClient(
    base_url="http://localhost:8100",
    api_key="sk-harness-...",
)

response = client.chat("gpt-4o", "What is Sagewai?")
print(response.content)
```

For the full SDK with agents, memory, workflows, and more:

```bash
pip install sagewai
```

## API Overview

### Harness (LLM Proxy)
- `chat(model, message)` -- Send a chat completion request through the harness
- `chat_stream(model, message)` -- Stream a chat completion response
- `list_models()` -- List available models

### Registry (Agent Governance)
- `list_agents()` -- List registered agents
- `get_agent(name)` -- Get agent detail
- `update_agent_config(name, config)` -- Update agent configuration

### Observatory (Cost Tracking)
- `get_spend(orgId)` -- Get spend summary
- `get_spend_breakdown(orgId)` -- Cost breakdown by model
- `get_audit_log(limit)` -- Query audit events
- `list_runs(agentName)` -- List agent runs

### Gateway (Token Management)
- `create_token(agentName)` -- Create access token
- `list_tokens()` -- List tokens
- `revoke_token(tokenId)` -- Revoke a token

## Authentication

```python
client = SagewaiClient(
    base_url="http://localhost:8100",
    api_key="sk-harness-...",
)
```

The API key is sent as a Bearer token in the Authorization header.


## Architecture & security model

This client is a **thin wrapper** over the Sagewai REST API. It does
not execute workflows itself — it submits enqueue requests to a
Sagewai control plane and queries run status. The server's worker
fleet does the actual work.

### Two credential tiers

Sagewai separates credentials into two tiers:

- **Tier-1 (orchestration)** — the LLM key the Sagewai Agent uses
  for planning and dispatch. Lives on the worker process. Operators
  manage it with their existing infrastructure. **Clients never see
  Tier-1 keys.**
- **Tier-2 (user-task)** — per-customer credentials (Anthropic API
  keys, GitHub tokens, AWS keys, customer database URLs) that CLI
  agents and tools use inside the sandbox. **Clients never see
  Tier-2 plaintext either** — clients only reference Sealed Identity
  profiles by name (`security_profile_ref`).

Your client code holds the Sagewai server's API key (which
authenticates it to the control plane) and a list of Sealed Identity
profile names. It never holds the credentials those profiles contain.

For the full architecture model — five execution modes (0 / 1 / 2 /
3 / 3b), the trust boundary, audit and revocation primitives — see:

- <https://docs.sagewai.ai/docs/architecture> — user-facing
- <https://github.com/sagewai/platform/tree/main/docs/architecture> — canonical contract

### Enqueue API surface

A workflow run is enqueued with:

- `input_data` — the workflow input
- `execution_mode` — Mode 0 (Bare) / 1 (Sandboxed) / 2 (Identity) /
  3 (Full + CLI agent) / 3b (Full + JIT callback, planned)
- `security_profile_ref` (optional) — name of the Sealed Identity
  profile to inject into the sandbox at start
- `artifact_destination` (optional, Mode 3+) — where CLI agent
  output goes (GitHub repo, S3 bucket, mounted folder)

This client may not yet expose all of these fields. See the
[client-libraries tracking issue](https://github.com/sagewai/platform/issues/166)
for rollout status across all 17 wrapper repos.

## Pillar Examples

These examples show how a Python developer uses the Sagewai REST API to get deterministic AI agent behavior in their own code.

### SDK Pillar -- Chat with an AI agent

```python
response = client.chat("gpt-4o",
    system="You are a legal analyst.",
    message="Summarize the key risks in this NDA.")
print(response.content)
print(f"Tokens: {response.total_tokens}")
```

### Harness Pillar -- Check team LLM spend

```python
spend = client.get_spend("acme-corp")
print(f"Monthly: ${spend.monthly_usd:.2f}")
```

### Registry Pillar -- List registered agents

```python
agents = client.list_agents()
for a in agents:
    print(f"{a.name} ({a.model})")
```

Every Sagewai pillar (SDK, Registry, Harness, Observatory, Training) is accessible via the REST API. The harness ensures deterministic model routing regardless of which LLM provider is used.

## Documentation

- [Sagewai Docs](https://docs.sagewai.ai)
- [OpenAPI Spec](https://github.com/sagewai/sagewai/blob/main/openapi/sagewai.yaml)
- [GitHub](https://github.com/sagewai/sagewai)

## License

AGPL-3.0. See [LICENSE](https://github.com/sagewai/sagewai/blob/main/LICENSE).

Built by [Ali Arda Diri](https://sagecurator.com)(https://sagewai.ai).
