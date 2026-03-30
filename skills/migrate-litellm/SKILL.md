---
description: Migrate from LiteLLM to Merge Gateway. Use when the user wants to replace LiteLLM proxy, switch from litellm.completion(), or move off a self-hosted LiteLLM setup.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

# Migrate LiteLLM to Merge Gateway

Migrate from a self-hosted LiteLLM proxy or the LiteLLM Python library to Merge Gateway.

## Language Support

The Merge Gateway SDK is available in both **Python** and **TypeScript/Node**:

- **Python:** `pip3 install merge-gateway-sdk`
- **TypeScript/Node:** `npm install merge-gateway-sdk`

Detect the user's stack and show the relevant language.

## Steps

### 0. Check for Plugin Updates

Before proceeding, ensure the user has the latest version of the Merge Gateway skills by running:
```
claude plugin update merge-gateway@merge-gateway-skills
```

**Run this update and wait for it to complete before continuing to Step 1.**
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

Check if `MERGE_GATEWAY` or `api-gateway.merge.dev` already exists in the project. If so, report which parts are already migrated and skip those.

### 3. Determine Migration Path

**Path A: LiteLLM Proxy Client** — The project uses the OpenAI SDK pointed at a LiteLLM proxy. This is the simpler migration.

**Path B: LiteLLM Python Library** — The project imports `litellm` directly and calls `litellm.completion()`. This requires replacing the library calls.

If the search results make it clear which path applies, proceed with that path. If ambiguous, ask the user: **"Are you using the LiteLLM proxy (OpenAI SDK pointed at localhost) or the LiteLLM Python library (importing litellm directly)?"** **STOP and wait for their response before proceeding.**

### 4A. Migrate LiteLLM Proxy Client

If the project uses the OpenAI SDK pointed at a LiteLLM proxy, just swap the URL and key.

**Choose your migration path:** Ask the user whether they want the quick migration (Option A) or the full migration (Option B).

**STOP here and wait for the user's response.** Do NOT proceed until they answer. Once they answer, follow ONLY the matching option below:

**Option A — Quick migration (keep OpenAI SDK):** Since LiteLLM proxy clients already use the OpenAI SDK, just change the base URL and API key.

Python:
```python
# Quick path — swap URL and key, all chat.completions.create() calls work unchanged
from openai import OpenAI
client = OpenAI(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url="https://api-gateway.merge.dev/v1/openai",
)
```

TypeScript:
```typescript
// Quick path — swap URL and key, all chat.completions.create() calls work unchanged
import OpenAI from "openai";
const client = new OpenAI({
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
  baseURL: "https://api-gateway.merge.dev/v1/openai",
});
```

**Option B — Full migration (native Merge Gateway SDK):** Switch to the Merge Gateway SDK for full access to tags, routing metadata, and model discovery.

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
});
```

**Model names:** LiteLLM accepts models in multiple formats. Gateway requires `provider/model` format. Handle each case:

**Already prefixed (transfer directly):**
- `openai/gpt-4o` → `openai/gpt-4o` (no change)
- `anthropic/claude-3-5-sonnet-20241022` → `anthropic/claude-3-5-sonnet-20241022` (no change)

**Un-prefixed models (need provider prefix added):**
- `gpt-4o` → `openai/gpt-4o`
- `gpt-4o-mini` → `openai/gpt-4o-mini`
- `claude-3-5-haiku-20241022` → `anthropic/claude-3-5-haiku-20241022`
- `claude-sonnet-4-6-20250514` → `anthropic/claude-sonnet-4-6-20250514`

**LiteLLM-specific prefixes (need remapping):**
- `bedrock/anthropic.claude-3-5-sonnet-20241022-v2:0` → `anthropic/claude-3-5-sonnet-20241022` (strip Bedrock prefix and ID format)
- `azure/my-deployment` → ask user what model the deployment maps to (e.g., `openai/gpt-4o`)

**IMPORTANT:** When models lack a provider prefix, always add one. Check the config, function call sites, and any variable where model names are defined.

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
)

response = client.responses.create(
    model="openai/gpt-4o",
    input=[{"type": "message", "role": "user", "content": "Hello!"}],
)
print(response.output[0].content[0].text)
```

**Async migration:**

> **Note:** The Merge Gateway Python SDK does not yet have an async client (`AsyncMergeGateway` is not available). For async code, wrap the synchronous client call in an executor:

```python
# Before
import litellm

response = await litellm.acompletion(
    model="openai/gpt-4o",
    messages=messages,
)

# After — use sync client in an executor until async is supported
import asyncio
import os
from merge_gateway import MergeGateway

client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
)

response = await asyncio.to_thread(
    client.responses.create,
    model="openai/gpt-4o",
    input=[{"type": "message", "role": msg["role"], "content": msg["content"]} for msg in messages],
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

# After — each chunk has accumulated text, print only new portion
response = client.responses.create(
    model="openai/gpt-4o",
    input=[{"type": "message", "role": msg["role"], "content": msg["content"]} for msg in messages],
    stream=True,
)
prev_text = ""
for chunk in response:
    if chunk.get("object") == "response.stream":
        content = chunk.get("output", [{}])[0].get("content", [])
        if content and content[0].get("type") == "text":
            new_text = content[0].get("text", "")
            print(new_text[len(prev_text):], end="", flush=True)
            prev_text = new_text
print()
```

**Embedding migration:**
```python
# Before
response = litellm.embedding(model="openai/text-embedding-ada-002", input=["Hello"])

# After
response = client.embeddings.create(model="openai/text-embedding-ada-002", input=["Hello"])
```

TypeScript:
```typescript
// Before (using OpenAI SDK pointed at LiteLLM)
const response = await client.embeddings.create({ model: "openai/text-embedding-ada-002", input: ["Hello"] });

// After (Merge Gateway)
const response = await client.embeddings.create({ model: "openai/text-embedding-ada-002", input: ["Hello"] });
```

### 5. Remove LiteLLM-Specific Configuration

Search for and remove LiteLLM-specific parameters:
- `litellm.set_verbose` — remove (Gateway has its own logging dashboard)
- `litellm.success_callback` / `litellm.failure_callback` — remove callbacks (Gateway handles observability)
- `litellm.api_base` — replaced by OpenAI client `base_url`
- `litellm.drop_params` — remove (Gateway handles parameter compatibility)
- `litellm.modify_params` — remove

### 6. Update Environment Variables

Old LiteLLM and provider keys are replaced by a single `MERGE_GATEWAY_API_KEY`.

**First, ask the user:** "Are you setting this up for **local development** or a **deployed environment**?"

**STOP here and wait for the user's response.** Do NOT proceed until they answer. Once they answer, follow ONLY the matching path below:

- **Local development:** Tell the user to run this in their terminal, replacing `mg_YOUR_KEY` with their actual key from [gateway.merge.dev](https://gateway.merge.dev):
  ```
  ! echo "MERGE_GATEWAY_API_KEY=mg_YOUR_KEY" >> .env
  ```
  Then verify `.gitignore` includes `.env`. Comment out (do NOT delete) old LiteLLM and provider keys in `.env`:
  ```
  # LITELLM_API_KEY=...               # Replaced by MERGE_GATEWAY_API_KEY
  # LITELLM_BASE_URL=...              # Replaced by MERGE_GATEWAY_API_KEY
  # OPENAI_API_KEY=...                # Replaced by MERGE_GATEWAY_API_KEY
  # ANTHROPIC_API_KEY=...             # Replaced by MERGE_GATEWAY_API_KEY
  ```

- **Deployed / CI/CD:** Tell the user to add `MERGE_GATEWAY_API_KEY` to their secrets manager or CI/CD environment variables, and remove the old keys (`LITELLM_API_KEY`, `LITELLM_BASE_URL`, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`) from their secrets configuration.

**Never** ask the user to paste their API key into the Claude conversation. Always give them a command or instructions they execute themselves.

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

Python (`test_gateway.py`):
```python
import os
from dotenv import load_dotenv
from merge_gateway import MergeGateway

load_dotenv()

client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
)

response = client.responses.create(
    model="openai/gpt-4o-mini",
    input=[{"type": "message", "role": "user", "content": "What is 2 + 2? Reply with just the number."}],
)
print("Gateway response:", response.output[0].content[0].text)
print("Model used:", response.model)
print("Migration verified successfully!")
```

TypeScript (`test_gateway.ts`):
```typescript
import dotenv from "dotenv";
import { MergeGateway } from "merge-gateway-sdk";

dotenv.config();

const client = new MergeGateway({
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
});

async function main() {
  const response = await client.responses.create({
    model: "openai/gpt-4o-mini",
    input: [{ type: "message", role: "user", content: "What is 2 + 2? Reply with just the number." }],
  });
  console.log("Gateway response:", response.output[0].content[0].text);
  console.log("Model used:", response.model);
  console.log("Migration verified successfully!");
}

main();
```

## Cross-Cutting Rules

- **Never delete old configuration** — comment out old env vars with a note about the replacement.
- **Never delete infrastructure without asking** — LiteLLM proxy Docker/K8s configs should be flagged, not removed.
- **Idempotency** — Check if migration is already partially applied before making changes.
- **Provider-prefixed models** — ALL model names must use `provider/model` format.
- **Base URL** — The SDK defaults to `https://api-gateway.merge.dev/v1`. Only pass `base_url`/`baseUrl` if the user has a custom gateway endpoint.
- **Remove `litellm` dependency last** — Only after all `litellm` imports are removed.
