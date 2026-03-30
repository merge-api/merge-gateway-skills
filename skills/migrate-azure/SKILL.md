---
description: Migrate from Azure OpenAI to Merge Gateway. Use when the user wants to switch from Azure OpenAI, replace AzureOpenAI SDK calls, or move off Azure deployments.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

# Migrate Azure OpenAI to Merge Gateway

Migrate from the Azure OpenAI SDK to the Merge Gateway SDK.

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

Replace `AzureOpenAI` with the Merge Gateway SDK.

**Choose your migration path:** Ask the user whether they want the quick migration (Option A) or the full migration (Option B).

**STOP here and wait for the user's response.** Do NOT proceed until they answer. Once they answer, follow ONLY the matching option below:

**Option A — Quick migration (keep OpenAI SDK):** Since Azure OpenAI uses the OpenAI SDK, you can keep it and just point at Gateway.

Python:
```python
# Quick path — keep OpenAI SDK, point at Gateway
from openai import OpenAI
client = OpenAI(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url="https://api-gateway.merge.dev/v1/openai",
)
# Replace deployment names with provider-prefixed models: "my-gpt4o" → "openai/gpt-4o"
```

TypeScript:
```typescript
// Quick path — keep OpenAI SDK, point at Gateway
import OpenAI from "openai";
const client = new OpenAI({
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
  baseURL: "https://api-gateway.merge.dev/v1/openai",
});
// Replace deployment names with provider-prefixed models: "my-gpt4o" → "openai/gpt-4o"
```

**Option B — Full migration (native Merge Gateway SDK):** Switch to the Merge Gateway SDK for full access to tags, routing metadata, and model discovery.

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
)
```

TypeScript:
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
)
```

The `azure-identity` dependency can be removed if only used for Azure OpenAI.

### 7. Update Environment Variables

Old Azure keys are replaced by a single `MERGE_GATEWAY_API_KEY`.

**First, ask the user:** "Are you setting this up for **local development** or a **deployed environment**?"

**STOP here and wait for the user's response.** Do NOT proceed until they answer. Once they answer, follow ONLY the matching path below:

- **Local development:** Tell the user to run this in their terminal, replacing `mg_YOUR_KEY` with their actual key from [gateway.merge.dev](https://gateway.merge.dev):
  ```
  ! echo "MERGE_GATEWAY_API_KEY=mg_YOUR_KEY" >> .env
  ```
  Then verify `.gitignore` includes `.env`. Comment out (do NOT delete) old Azure keys in `.env`:
  ```
  # AZURE_OPENAI_API_KEY=...          # Replaced by MERGE_GATEWAY_API_KEY
  # AZURE_OPENAI_ENDPOINT=...         # Replaced by MERGE_GATEWAY_API_KEY
  # AZURE_OPENAI_API_VERSION=...      # Replaced by MERGE_GATEWAY_API_KEY
  # AZURE_OPENAI_DEPLOYMENT=...       # Replaced by MERGE_GATEWAY_API_KEY
  ```

- **Deployed / CI/CD:** Tell the user to add `MERGE_GATEWAY_API_KEY` to their secrets manager or CI/CD environment variables, and remove the old Azure keys (`AZURE_OPENAI_API_KEY`, `AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_API_VERSION`, `AZURE_OPENAI_DEPLOYMENT`) from their secrets configuration.

**Never** ask the user to paste their API key into the Claude conversation. Always give them a command or instructions they execute themselves.

### 8. Verify

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

**Embeddings migration:**

Azure embeddings migrate the same way — just prefix the model name:

```python
# Before (Azure)
response = client.embeddings.create(model="text-embedding-ada-002", input="Hello")

# After (Merge Gateway)
response = client.embeddings.create(model="openai/text-embedding-ada-002", input="Hello")
# Response format is the same: response.data[0].embedding
```

TypeScript:
```typescript
// Before (Azure)
const response = await client.embeddings.create({ model: "text-embedding-ada-002", input: "Hello" });

// After (Merge Gateway)
const response = await client.embeddings.create({ model: "openai/text-embedding-ada-002", input: "Hello" });
// Response format is the same: response.data[0].embedding
```

## Cross-Cutting Rules

- **Never delete old configuration** — comment out old env vars with a note about the replacement.
- **Idempotency** — Check if migration is already partially applied before making changes.
- **Provider-prefixed models** — ALL model names must use `provider/model` format.
- **Base URL** — The SDK defaults to `https://api-gateway.merge.dev/v1`. Only pass `base_url`/`baseUrl` if the user has a custom gateway endpoint.
- **Azure deployment names are opaque** — Always ask the user to confirm what model each deployment maps to.
