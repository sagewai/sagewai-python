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
