# Sagewai Python API Client

> **Status: placeholder — this client library is not yet published.**
> This repository currently holds the API-client specification and documentation for the
> Sagewai Python client. The package described below is **not yet available on a package
> registry**; the code samples show the intended API. The Go, Rust, and TypeScript clients
> are the implemented, verified clients today — see [docs.sagewai.ai](https://docs.sagewai.ai)
> for status and release updates.

Official Python API client for the [Sagewai](https://sagewai.ai) agent infrastructure platform.

This is a **thin RPC wrapper**. It does not execute work — it submits enqueue requests to a Sagewai control plane and queries run status. The worker fleet is the only executor; the wrapper never holds Tier-2 secrets.

> Architecture: see [runtime topology](https://docs.sagewai.ai/docs/architecture/runtime-topology), [security tiers](https://docs.sagewai.ai/docs/architecture/security-tiers), [execution modes](https://docs.sagewai.ai/docs/architecture/execution-modes), [execution backends](https://docs.sagewai.ai/docs/architecture/sandbox-backends). Those docs are the contract.

The full Sagewai Python SDK — agents, memory, workflows, and more — plus this API client are published as the single `sagewai` package (`pip install sagewai`). There is no separate `sagewai-client` package.

## Security model

Sagewai has two credential tiers (see [security-tiers.md](https://docs.sagewai.ai/docs/architecture/security-tiers)). The wrapper sees **neither** in plaintext:

- **Tier 1** — orchestration keys, in worker process env. The wrapper never references these.
- **Tier 2** — user-task keys (Claude Code, GitHub tokens, AWS), in the sandbox's environment variables only. The wrapper only references Sealed *profile refs* (e.g. `acme-prod`).

`WorkflowRun.effective_env_keys` and `effective_secret_keys` are **names only** — never values.

## Execution modes

Per [execution-modes.md](https://docs.sagewai.ai/docs/architecture/execution-modes):

- `BARE` — Mode 0 — pure orchestration on the worker, no sandbox.
- `SANDBOXED` — Mode 1 — untrusted code, no customer creds.
- `IDENTITY` — Mode 2 — sandbox + Sealed identity (DB / API / S3 with customer creds).
- `FULL` — Mode 3 — CLI agent (Claude Code, Codex) + artifact destination.
- `FULL_JIT` — Mode 3b — Mode 3 + JIT credential callbacks.

The wrapper exposes all five via the `ExecutionMode` enum.

## Install

```bash
pip install sagewai
```

Requires Python 3.10+.

## Quick Start (Mode 0 — planning)

```python
from sagewai import SagewaiClient, EnqueueRequest, ExecutionMode

client = SagewaiClient(base_url="http://localhost:8000", api_key="sk-...")

resp = client.enqueue_run("summarise-brief", EnqueueRequest(
    input_data={"brief": "Quarterly OKR draft for engineering"},
    execution_mode=ExecutionMode.BARE,
))
print(f"enqueued run {resp.run_id} ({resp.status})")
```

## Mode 3 example — CLI agent (Claude Code building a portfolio site)

```python
from sagewai import ArtifactDestination

resp = client.enqueue_run("build-portfolio-site", EnqueueRequest(
    input_data={"brief": "minimalist Next.js portfolio"},
    execution_mode=ExecutionMode.FULL,
    security_profile_ref="customer-acme",
    artifact_destination=ArtifactDestination(type="github", target="acme/portfolio"),
))
print(f"enqueued Mode 3 run {resp.run_id}")
```

## Polling for completion

```python
run = client.get_workflow_run(resp.run_id)
print(f"status: {run.status} | secret keys (names only): {run.effective_secret_keys}")
```

## Legacy harness API

Chat / agent registry / spend / token methods still exist on the client unchanged. The reference below documents them, but a few signatures and types in the legacy sections may have drifted from the actual client surface and will be reconciled in a follow-up. The Pillar Examples further down reflect the current API.

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
    base_url="http://localhost:8000",
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
- [REST API reference](https://docs.sagewai.ai/docs/api-reference/rest-api)
- [Sagewai docs](https://docs.sagewai.ai)

## Disclaimer

This software is provided **"AS IS", without warranty of any kind**, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and noninfringement. In no event shall Sagecurator or any contributor be liable for any claim, damages, or other liability, whether in an action of contract, tort, or otherwise, arising from, out of, or in connection with the software or the use or other dealings in the software. You use it entirely at your own risk. See [LICENSE](LICENSE) (AGPL-3.0, sections 15 and 16) for the full warranty disclaimer and limitation of liability.

## License

AGPL-3.0. See [LICENSE](LICENSE).

Built by [Sagecurator](https://sagewai.ai).
