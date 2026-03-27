---
description: Migrate from Azure OpenAI to Merge Gateway. Use when the user wants to switch from Azure OpenAI, replace AzureOpenAI SDK calls, or move off Azure deployments.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

# Migrate Azure OpenAI to Merge Gateway

Migrate from the Azure OpenAI SDK to the Merge Gateway SDK.

## Language Support

**The Python SDK (`merge-gateway-sdk`) is the default and primary Gateway SDK.** Always prefer Python examples and migration paths. The TypeScript/Node SDK is **coming soon** and not yet published. TypeScript examples are included below for reference and future use only.

## Steps

### 1. Search for Azure OpenAI Usage

Search the project for all Azure OpenAI references:

- **Imports:** `from openai import AzureOpenAI`, `import { AzureOpenAI }`, `AzureOpenAI`
- **Constructors:** `AzureOpenAI(`, `new AzureOpenAI({`
- **Parameters:** `azure_endpoint`, `api_version`, `azure_deployment`, `azure_ad_token`
- **Env vars:** `AZURE_OPENAI_API_KEY`, `AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_API_VERSION`, `AZURE_OPENAI_DEPLOYMENT`, `AZURE_API_KEY`
- **Config files:** `.env`, `.env.example`, `appsettings.json`, `docker-compose.yml`

Report all findings to the user before making changes.

### 2. Check for Prior Migration

Check if `MERGE_GATEWAY` or `api-gateway.merge.dev` already exists in the project. If so, report which parts are already migrated and skip those.

### 3. Map Azure Deployments to Gateway Model Names

Azure uses custom deployment names that map to underlying models. Ask the user what model each deployment name maps to:

Common mappings:
| Azure Deployment (example) | Gateway Model Name |
|---|---|
| `gpt-4o` / `my-gpt4o-deployment` | `openai/gpt-4o` |
| `gpt-4o-mini` / `my-gpt4o-mini` | `openai/gpt-4o-mini` |
| `gpt-4-turbo` / `my-gpt4-turbo` | `openai/gpt-4-turbo` |
| `gpt-35-turbo` / `my-gpt35` | `openai/gpt-3.5-turbo` |
| `text-embedding-ada-002` | `openai/text-embedding-ada-002` |

**IMPORTANT:** Azure deployment names are user-defined and may not match the model name. Always ask the user to confirm the mapping.

### 4. Migrate Client Constructors

Replace `AzureOpenAI` with the Merge Gateway SDK:

Python:
```python
# Before
from openai import AzureOpenAI

client = AzureOpenAI(
    azure_endpoint=os.environ["AZURE_OPENAI_ENDPOINT"],
    api_key=os.environ["AZURE_OPENAI_API_KEY"],
    api_version="2024-02-01",
)

# After
from merge_gateway import MergeGateway

client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
)
```

TypeScript (coming soon — SDK not yet published):
```typescript
// Before
import { AzureOpenAI } from "openai";

const client = new AzureOpenAI({
  endpoint: process.env.AZURE_OPENAI_ENDPOINT,
  apiKey: process.env.AZURE_OPENAI_API_KEY,
  apiVersion: "2024-02-01",
  deployment: "my-gpt4o-deployment",
});

// After
import { MergeGateway } from "merge-gateway-sdk";

const client = new MergeGateway({
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
  baseUrl: process.env.MERGE_GATEWAY_BASE_URL + "/v1",
});
```

### 5. Update API Calls

Replace Azure deployment names with Gateway model names in all API calls:

Python:
```python
# Before
response = client.chat.completions.create(
    model="my-gpt4o-deployment",  # Azure deployment name
    messages=[{"role": "user", "content": "Hello!"}],
)

# After
response = client.responses.create(
    model="openai/gpt-4o",  # Gateway provider-prefixed model
    input=[{"type": "message", "role": "user", "content": "Hello!"}],
)
```

If the code passes `model` as a variable, trace back to where the value is defined and update it there.

### 6. Remove Azure-Specific Parameters

Remove all Azure-specific configuration:
- `api_version` — not needed with Gateway
- `azure_deployment` — replaced by model name in API calls
- `azure_endpoint` — replaced by Gateway base URL
- `azure_ad_token` / `azure_ad_token_provider` — replaced by Gateway API key

If using Azure AD token authentication:
```python
# Before
from azure.identity import DefaultAzureCredential, get_bearer_token_provider

credential = DefaultAzureCredential()
token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")

client = AzureOpenAI(
    azure_endpoint=os.environ["AZURE_OPENAI_ENDPOINT"],
    azure_ad_token_provider=token_provider,
)

# After
from merge_gateway import MergeGateway

client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
)
```

The `azure-identity` dependency can be removed if only used for Azure OpenAI.

### 7. Update Environment Variables

```
# Before
AZURE_OPENAI_API_KEY=...
AZURE_OPENAI_ENDPOINT=https://my-resource.openai.azure.com
AZURE_OPENAI_API_VERSION=2024-02-01
AZURE_OPENAI_DEPLOYMENT=my-gpt4o-deployment

# After
MERGE_GATEWAY_API_KEY=mg_your_api_key_here
MERGE_GATEWAY_BASE_URL=https://api-gateway.merge.dev
# AZURE_OPENAI_API_KEY=...                               # Replaced by MERGE_GATEWAY_API_KEY
# AZURE_OPENAI_ENDPOINT=https://my-resource.openai.azure.com  # Replaced by MERGE_GATEWAY_BASE_URL
# AZURE_OPENAI_API_VERSION=2024-02-01                    # Not needed with Gateway
# AZURE_OPENAI_DEPLOYMENT=my-gpt4o-deployment            # Use model name directly
```

Comment out (do NOT delete) old Azure env vars.

### 8. Verify

Generate a test script:

Python (`test_gateway.py`):
```python
import os
from merge_gateway import MergeGateway

client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
)

response = client.responses.create(
    model="openai/gpt-4o",
    input=[{"type": "message", "role": "user", "content": "Say 'Azure migration successful!' and nothing else."}],
)
print(response.output[0].content[0].text)
```

TypeScript (`test_gateway.ts`) — coming soon, SDK not yet published:
```typescript
import { MergeGateway } from "merge-gateway-sdk";

const client = new MergeGateway({
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
  baseUrl: process.env.MERGE_GATEWAY_BASE_URL + "/v1",
});

async function main() {
  const response = await client.responses.create({
    model: "openai/gpt-4o",
    input: [{ type: "message", role: "user", content: "Say 'Azure migration successful!' and nothing else." }],
  });
  console.log(response.output[0].content[0].text);
}

main();
```

**Embeddings migration:**

Azure embeddings migrate the same way — just prefix the model name:

```python
# Before (Azure)
response = client.embeddings.create(model="text-embedding-ada-002", input="Hello")

# After (Merge Gateway)
response = client.embeddings.create(model="openai/text-embedding-ada-002", input="Hello")
# Response format is the same: response.data[0].embedding
```

## Cross-Cutting Rules

- **Never delete old configuration** — comment out old env vars with a note about the replacement.
- **Idempotency** — Check if migration is already partially applied before making changes.
- **Provider-prefixed models** — ALL model names must use `provider/model` format.
- **Base URL** — The env var `MERGE_GATEWAY_BASE_URL` should be set **without** `/v1` (e.g., `https://api-gateway.merge.dev`). Always append `/v1` in code. If the env var already contains `/v1`, do NOT append it again — check for this to avoid a double `/v1` path.
- **Azure deployment names are opaque** — Always ask the user to confirm what model each deployment maps to.
