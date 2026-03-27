---
description: Explore Merge Gateway advanced features including tags, project tracking, routing metadata, multi-model routing, and extended thinking. Use after initial setup to unlock Gateway's full capabilities.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

# Merge Gateway Advanced Features

Walk the user through Gateway's advanced features that differentiate it from calling providers directly. This skill assumes the user has already completed initial integration via `gateway-implement` or one of the migration skills.

## Language Support

**The Python SDK (`merge-gateway-sdk`) is the default and primary Gateway SDK.** The TypeScript/Node SDK is **coming soon** and not yet published.

## Steps

### 1. Verify Existing Integration

Before showing advanced features, confirm the user has a working Gateway setup:
- Check for `MERGE_GATEWAY_API_KEY` and `MERGE_GATEWAY_BASE_URL` in `.env`
- Check for `merge-gateway-sdk` in dependencies
- Check for `MergeGateway` client usage in code

If not set up, direct them to run the `gateway-implement` skill first.

### 2. Tags — Organize and Track API Calls

Tags let you label requests for filtering in the Gateway dashboard. This is useful for tracking costs by feature, team, or environment.

```python
from merge_gateway import MergeGateway
import os

client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
)

# Tag requests for tracking
response = client.responses.create(
    model="openai/gpt-4o",
    input=[{"type": "message", "role": "user", "content": "Summarize this document."}],
    tags=[
        {"key": "feature", "value": "document-summary"},
        {"key": "team", "value": "backend"},
        {"key": "environment", "value": "production"},
    ],
)
print(response.output[0].content[0].text)
```

**Common tag patterns:**
- `feature` — which product feature made this call (e.g., "chat", "search", "summarization")
- `team` — which team owns this code path
- `environment` — dev/staging/production
- `user_tier` — free/pro/enterprise (for per-tier cost analysis)
- `experiment` — A/B test variant names

You can also list available tags in your account:
```python
tags = client.tags.list()
for tag in tags.tags:
    print(f"{tag.key}: {tag.value}")
```

### 3. Project ID — Group Related Requests

Use `project_id` to group all API calls belonging to a specific project or workspace. This enables per-project usage tracking and budgeting in the dashboard.

```python
response = client.responses.create(
    model="openai/gpt-4o",
    input=[{"type": "message", "role": "user", "content": "Hello!"}],
    project_id="proj_customer_support_bot",
)
```

### 4. Routing Metadata — Understand Routing Decisions

Gateway can return metadata about how it routed your request. This is invaluable for debugging, cost analysis, and understanding which model/provider actually served your request.

```python
response = client.responses.create(
    model="openai/gpt-4o",
    input=[{"type": "message", "role": "user", "content": "Hello!"}],
    include_routing_metadata=True,
)

# Access routing details
if response.routing:
    print(f"Policy: {response.routing.policy_name}")
    print(f"Model requested: {response.routing.model_requested}")
    print(f"Model used: {response.routing.model_used}")
    print(f"Cost: ${response.routing.cost_usd:.6f}")
    print(f"Strategy: {response.routing.strategy}")

    if response.routing.intelligent_routing_used:
        print(f"Complexity score: {response.routing.complexity_score}")
        print(f"Selected tier: {response.routing.selected_tier}/{response.routing.total_tiers}")
        print(f"Routing reason: {response.routing.routing_reason}")

    if response.routing.latency:
        print(f"Total latency: {response.routing.latency.total_ms:.0f}ms")
        print(f"  Policy lookup: {response.routing.latency.policy_lookup_ms:.0f}ms")
        print(f"  Routing decision: {response.routing.latency.routing_decision_ms:.0f}ms")
        print(f"  LLM call: {response.routing.latency.llm_call_ms:.0f}ms")
```

**Routing metadata fields:**
| Field | Description |
|---|---|
| `policy_name` | Which routing policy was applied |
| `strategy` | Routing strategy used (e.g., "intelligent", "fallback") |
| `model_requested` | The model you asked for |
| `model_used` | The model that actually served the request (may differ with routing policies) |
| `cost_usd` | Cost of this specific request in USD |
| `intelligent_routing_used` | Whether the complexity-based router was used |
| `complexity_score` | How complex the request was rated (if intelligent routing) |
| `latency` | Breakdown of time spent in each phase |

### 5. Multi-Model Routing — One Client, Many Providers

The core value of Gateway: call any provider through the same client. Just change the model string.

```python
client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
)

question = "What is the capital of France?"
input_msg = [{"type": "message", "role": "user", "content": question}]

# Same client, different providers
openai_response = client.responses.create(model="openai/gpt-4o", input=input_msg)
anthropic_response = client.responses.create(model="anthropic/claude-sonnet-4-20250514", input=input_msg)
google_response = client.responses.create(model="google/gemini-2.0-flash", input=input_msg)

print("OpenAI:", openai_response.output[0].content[0].text)
print("Anthropic:", anthropic_response.output[0].content[0].text)
print("Google:", google_response.output[0].content[0].text)

# Compare costs with routing metadata
for model in ["openai/gpt-4o", "anthropic/claude-sonnet-4-20250514", "google/gemini-2.0-flash"]:
    resp = client.responses.create(model=model, input=input_msg, include_routing_metadata=True)
    cost = resp.routing.cost_usd if resp.routing else "N/A"
    print(f"{model}: ${cost}")
```

### 6. Browse Available Models

List all models available through Gateway:

```python
# List all available models
models = client.models.list()
for model in models.models:
    print(f"{model.id} — {model.provider}")

# Filter by provider
openai_models = client.models.list(provider="openai")
for model in openai_models.models:
    print(f"  {model.id}")

# Filter by capability
tool_models = client.models.list(supports_tool_calling=True)

# Get details for a specific model
model = client.models.retrieve("openai/gpt-4o")
print(f"Model: {model.id}")
print(f"Provider: {model.provider}")
if model.pricing:
    print(f"Input: ${model.pricing.input_per_million_tokens}/M tokens")
    print(f"Output: ${model.pricing.output_per_million_tokens}/M tokens")
```

### 7. Extended Thinking (Anthropic Claude)

For complex reasoning tasks, enable extended thinking on supported models:

```python
response = client.responses.create(
    model="anthropic/claude-sonnet-4-20250514",
    input=[{"type": "message", "role": "user", "content": "Solve this step by step: What is 127 * 843?"}],
    thinking={"type": "enabled", "budget_tokens": 5000},
)

# Check for thinking blocks in output
for block in response.output[0].content:
    if block.type == "thinking":
        print("Thinking:", block.thinking[:200], "...")
    elif block.type == "text":
        print("Answer:", block.text)
```

### 8. Token Usage Tracking

Every response includes token usage data:

```python
response = client.responses.create(
    model="openai/gpt-4o",
    input=[{"type": "message", "role": "user", "content": "Hello!"}],
)

if response.usage:
    print(f"Input tokens: {response.usage.input_tokens}")
    print(f"Output tokens: {response.usage.output_tokens}")
    print(f"Total tokens: {response.usage.total_tokens}")
```

### 9. Embeddings

Generate embeddings through Gateway with any supported embedding model:

```python
# Single text
response = client.embeddings.create(
    model="openai/text-embedding-3-small",
    input="The quick brown fox jumps over the lazy dog.",
)
print(f"Dimensions: {len(response.data[0].embedding)}")

# Batch embeddings
response = client.embeddings.create(
    model="openai/text-embedding-3-small",
    input=["First sentence.", "Second sentence.", "Third sentence."],
)
for i, item in enumerate(response.data):
    print(f"Text {i}: {len(item.embedding)} dimensions")

print(f"Total tokens: {response.usage.total_tokens}")
```

## What to Configure in the Dashboard

After learning these SDK features, direct the user to the [Gateway dashboard](https://app.merge.dev) to configure:

- **Routing Policies** — Set up fallback chains, load balancing across providers, and cost-optimized routing
- **Budgets** — Set spending limits per API key, project, or tag
- **Usage Analytics** — View request volumes, costs, and latency breakdowns by model, tag, or project
- **API Key Management** — Create and rotate API keys with different permissions

These dashboard features complement the SDK features above. Tags and project IDs you set in code become filterable dimensions in the dashboard.

## Cross-Cutting Rules

- **Provider-prefixed models** — ALL model names must use `provider/model` format.
- **Base URL** — The env var `MERGE_GATEWAY_BASE_URL` should be set **without** `/v1` (e.g., `https://api-gateway.merge.dev`). Always append `/v1` in code. If the env var already contains `/v1`, do NOT append it again — check for this to avoid a double `/v1` path.
- **Tags are optional** — Don't add tags to every call. Suggest them where they add value (production code, cost tracking, multi-team projects).
- **Routing metadata adds latency** — Only use `include_routing_metadata=True` when debugging or during development. Don't leave it on in production unless needed.
