# Migrate LiteLLM to Merge Gateway

Migrate from a self-hosted LiteLLM proxy or the LiteLLM Python library to Merge Gateway.

## Steps

### 1. Search for LiteLLM Usage

Search the project for all LiteLLM references:

**LiteLLM Proxy (most common):**
- URLs: `localhost:4000`, `localhost:8000` (common LiteLLM proxy ports), any URL with `/chat/completions` that points to a local/internal service
- Env vars: `LITELLM_API_KEY`, `LITELLM_BASE_URL`, `LITELLM_PROXY_URL`
- Config: `litellm_config.yaml`, `config.yaml` with `model_list`, `litellm_settings`

**LiteLLM Python Library (direct):**
- Imports: `import litellm`, `from litellm import completion`, `from litellm import acompletion`
- Function calls: `litellm.completion(`, `litellm.acompletion(`, `litellm.embedding(`
- Dependencies: `litellm` in `requirements.txt`, `pyproject.toml`, `Pipfile`

**LiteLLM Infrastructure:**
- Docker: `docker-compose.yml` with `litellm` service
- Kubernetes: `deployment.yaml`, `service.yaml` with litellm images
- Config files: `litellm_config.yaml`, `proxy_config.yaml`

Report all findings to the user before making changes.

### 2. Check for Prior Migration

Check if `MERGE_GATEWAY` or `gateway.merge.dev` already exists in the project. If so, report which parts are already migrated and skip those.

### 3. Determine Migration Path

**Path A: LiteLLM Proxy Client** — The project uses the OpenAI SDK pointed at a LiteLLM proxy. This is the simpler migration.

**Path B: LiteLLM Python Library** — The project imports `litellm` directly and calls `litellm.completion()`. This requires replacing the library calls.

Ask the user which path applies, or determine from the search results.

### 4A. Migrate LiteLLM Proxy Client

If the project uses the OpenAI SDK pointed at a LiteLLM proxy, just swap the URL and key:

Python:
```python
# Before
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:4000/v1",  # LiteLLM proxy
    api_key=os.environ["LITELLM_API_KEY"],
)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello!"}],
)

# After
from merge_gateway import MergeGateway

client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
)

response = client.responses.create(
    model="openai/gpt-4o",
    input=[{"type": "message", "role": "user", "content": "Hello!"}],
)
```

TypeScript:
```typescript
// Before
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "http://localhost:4000/v1",
  apiKey: process.env.LITELLM_API_KEY,
});

// After
import { MergeGateway } from "merge-gateway-sdk";

const client = new MergeGateway({
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
  baseUrl: process.env.MERGE_GATEWAY_BASE_URL + "/v1",
});
```

**Model names:** LiteLLM uses `provider/model` format — same as Gateway. Most names transfer directly. Check and update any that need it:
- `openai/gpt-4o` → `openai/gpt-4o` (no change)
- `anthropic/claude-3-5-sonnet-20241022` → `anthropic/claude-sonnet-4-20250514` (update to latest)
- `bedrock/anthropic.claude-3-5-sonnet-20241022-v2:0` → `anthropic/claude-sonnet-4-20250514` (different prefix)
- `azure/my-deployment` → `openai/gpt-4o` (ask user for mapping)

### 4B. Migrate LiteLLM Python Library

Replace `litellm.completion()` calls with Merge Gateway SDK calls:

```python
# Before
import litellm

response = litellm.completion(
    model="openai/gpt-4o",
    messages=[{"role": "user", "content": "Hello!"}],
    temperature=0.7,
)
print(response.choices[0].message.content)

# After
from merge_gateway import MergeGateway
import os

client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
)

response = client.responses.create(
    model="openai/gpt-4o",
    input=[{"type": "message", "role": "user", "content": "Hello!"}],
)
print(response.output[0].content)
```

**Async migration:**
```python
# Before
import litellm

response = await litellm.acompletion(
    model="openai/gpt-4o",
    messages=messages,
)

# After
from merge_gateway import AsyncMergeGateway
import os

client = AsyncMergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
)

response = await client.responses.create(
    model="openai/gpt-4o",
    input=[{"type": "message", "role": "user", "content": msg} for msg in messages],
)
```

**Streaming migration:**
```python
# Before
response = litellm.completion(
    model="openai/gpt-4o",
    messages=messages,
    stream=True,
)
for chunk in response:
    print(chunk.choices[0].delta.content or "", end="")

# After
response = client.responses.create(
    model="openai/gpt-4o",
    input=[{"type": "message", "role": "user", "content": msg} for msg in messages],
    stream=True,
)
for event in response:
    print(event, end="")
```

**Embedding migration:**
```python
# Before
response = litellm.embedding(model="openai/text-embedding-ada-002", input=["Hello"])

# After
response = client.responses.create(model="openai/text-embedding-ada-002", input=["Hello"])
```

### 5. Remove LiteLLM-Specific Configuration

Search for and remove LiteLLM-specific parameters:
- `litellm.set_verbose` — remove (Gateway has its own logging dashboard)
- `litellm.success_callback` / `litellm.failure_callback` — remove callbacks (Gateway handles observability)
- `litellm.api_base` — replaced by OpenAI client `base_url`
- `litellm.drop_params` — remove (Gateway handles parameter compatibility)
- `litellm.modify_params` — remove

### 6. Update Environment Variables

```
# Before
LITELLM_API_KEY=sk-...
LITELLM_BASE_URL=http://localhost:4000
# Provider keys that LiteLLM used:
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# After
MERGE_GATEWAY_API_KEY=mg_your_api_key_here
MERGE_GATEWAY_BASE_URL=https://gateway.merge.dev
# LITELLM_API_KEY=sk-...              # Replaced by MERGE_GATEWAY_API_KEY
# LITELLM_BASE_URL=http://localhost:4000  # Replaced by MERGE_GATEWAY_BASE_URL
# OPENAI_API_KEY=sk-...               # No longer needed — Gateway manages provider keys
# ANTHROPIC_API_KEY=sk-ant-...        # No longer needed — Gateway manages provider keys
```

Comment out (do NOT delete) old env vars.

### 7. Clean Up LiteLLM Infrastructure

Ask the user before removing any infrastructure. Flag these for potential cleanup:
- `litellm` dependency in `requirements.txt` / `pyproject.toml`
- LiteLLM proxy Docker service in `docker-compose.yml`
- LiteLLM Kubernetes deployments / services
- `litellm_config.yaml` / `proxy_config.yaml`

Do NOT delete these automatically — inform the user and let them decide.

### 8. Feature Mapping

Explain to the user how LiteLLM features map to Gateway:

| LiteLLM Feature | Merge Gateway Equivalent |
|---|---|
| `config.yaml` model routing | Routing policies (configured in Gateway dashboard) |
| Fallback models (`fallbacks` in config) | Fallback strategies (configured in Gateway dashboard) |
| Rate limiting (`max_parallel_requests`) | Automatic rate limiting |
| Spend tracking / budgets | Usage dashboard + budget controls |
| `success_callback` / logging | Gateway logs + analytics dashboard |
| Load balancing (`model_list` with multiple deployments) | Routing policies with load balancing |
| API key management (`/key/generate`) | Gateway API key management (dashboard or API) |

### 9. Verify

Generate a test script:

```python
import os
from merge_gateway import MergeGateway

client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
)

response = client.responses.create(
    model="openai/gpt-4o",
    input=[{"type": "message", "role": "user", "content": "Say 'LiteLLM migration successful!' and nothing else."}],
)
print(response.output[0].content)
```

## Cross-Cutting Rules

- **Never delete old configuration** — comment out old env vars with a note about the replacement.
- **Never delete infrastructure without asking** — LiteLLM proxy Docker/K8s configs should be flagged, not removed.
- **Idempotency** — Check if migration is already partially applied before making changes.
- **Provider-prefixed models** — ALL model names must use `provider/model` format.
- **Merge Gateway SDK base URL** — Always append `/v1`: `os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1"`.
- **Remove `litellm` dependency last** — Only after all `litellm` imports are removed.
