---
description: Add Merge Gateway to an existing project. Use when the user wants to integrate, set up, or add Gateway to their codebase.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

# Integrate Merge Gateway

Guide a developer through adding Merge Gateway to an existing project. Detect their stack, install the Merge Gateway SDK, update configuration, and verify the integration works.

## Steps

### 1. Detect Stack

Search the project for existing LLM SDK usage:

- **Python:** Check `requirements.txt`, `pyproject.toml`, `Pipfile`, `setup.py` for `openai`, `anthropic`, `langchain`, `litellm`, `boto3`, `google-generativeai`
- **TypeScript/Node:** Check `package.json` for `openai`, `@anthropic-ai/sdk`, `langchain`, `litellm`, `@google/generative-ai`, `@azure/openai`

Search for existing LLM client constructors:
- `OpenAI(`, `new OpenAI({`, `Anthropic(`, `new Anthropic({`, `AzureOpenAI(`, `boto3.client('bedrock`

Report what you found to the user before proceeding.

### 2. Ask for Credentials

Ask the user for:
- **Gateway API key** (`mg_...`) — or ask if they want to use an env var (default: `MERGE_GATEWAY_API_KEY`)
- **Gateway base URL** — default: `https://api-gateway.merge.dev`

### 3. Install the Merge Gateway SDK

Python:
```bash
pip install merge-gateway-sdk
```

TypeScript/Node:
```bash
npm install merge-gateway-sdk
```

### 4. Update SDK Configuration

**Primary path — Merge Gateway SDK:**

Replace existing LLM client constructors with the Merge Gateway SDK:

Python:
```python
# Before (OpenAI)
from openai import OpenAI
client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])

# After (Merge Gateway)
from merge_gateway import MergeGateway

client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
)
```

TypeScript:
```typescript
// Before (OpenAI)
import OpenAI from "openai";
const client = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

// After (Merge Gateway)
import { MergeGateway } from "merge-gateway-sdk";

const client = new MergeGateway({
  apiKey: process.env.MERGE_GATEWAY_API_KEY,
  baseUrl: process.env.MERGE_GATEWAY_BASE_URL + "/v1",
});
```

**Update API calls** to use the Merge Gateway unified format:

Python:
```python
# Before (OpenAI)
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are helpful."},
        {"role": "user", "content": "Hello!"},
    ],
)
print(response.choices[0].message.content)

# After (Merge Gateway)
response = client.responses.create(
    model="openai/gpt-4o",
    input=[
        {"type": "message", "role": "system", "content": "You are helpful."},
        {"type": "message", "role": "user", "content": "Hello!"},
    ],
)
print(response.output[0].content[0].text)
```

TypeScript:
```typescript
// Before (OpenAI)
const response = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [
    { role: "system", content: "You are helpful." },
    { role: "user", content: "Hello!" },
  ],
});
console.log(response.choices[0].message.content);

// After (Merge Gateway)
const response = await client.responses.create({
  model: "openai/gpt-4o",
  input: [
    { type: "message", role: "system", content: "You are helpful." },
    { type: "message", role: "user", content: "Hello!" },
  ],
});
console.log(response.output[0].content[0].text);
```

Update model names to provider-prefixed format:
- `gpt-4o` → `openai/gpt-4o`
- `gpt-4o-mini` → `openai/gpt-4o-mini`
- `gpt-4-turbo` → `openai/gpt-4-turbo`
- `gpt-3.5-turbo` → `openai/gpt-3.5-turbo`
- `claude-sonnet-4-20250514` → `anthropic/claude-sonnet-4-20250514`
- `claude-3-5-haiku-20241022` → `anthropic/claude-3-5-haiku-20241022`
- `gemini-2.0-flash` → `google/gemini-2.0-flash`

**Alternative — OpenAI SDK compatibility (for codebases that prefer minimal changes):**

If the user prefers to keep the OpenAI SDK, they can point it at Gateway instead:

Python:
```python
from openai import OpenAI

client = OpenAI(
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1/openai",
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
)
```

TypeScript:
```typescript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: process.env.MERGE_GATEWAY_BASE_URL + "/v1/openai",
  apiKey: process.env.MERGE_GATEWAY_API_KEY,
});
```

This uses the standard `chat.completions.create()` API but still routes through Gateway. Model names must still use the `provider/model` format.

### 5. Detect Centralized Config

Before making scattered changes, search for centralized configuration patterns:
- Python: `config.py`, `settings.py`, `.env` loading, `pydantic.BaseSettings`
- TypeScript: `config.ts`, `settings.ts`, DI containers, `process.env` centralization

If a centralized config exists, update the Gateway settings there and have call sites reference the config. If not, update each call site directly.

### 6. Set Up Environment Variables

Create or update `.env` (and `.env.example` if it exists):
```
MERGE_GATEWAY_API_KEY=mg_your_api_key_here
MERGE_GATEWAY_BASE_URL=https://api-gateway.merge.dev
```

Check `.gitignore` includes `.env`. If not, warn the user.

Comment out (do NOT delete) any old provider API key env vars:
```
# OPENAI_API_KEY=sk-...  # Replaced by MERGE_GATEWAY_API_KEY
```

### 7. Verify Integration

Generate a test script that makes one call through Gateway:

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
    input=[
        {"type": "message", "role": "user", "content": "Say 'Gateway integration successful!' and nothing else."},
    ],
)
print(response.output[0].content[0].text)
```

TypeScript (`test_gateway.ts`):
```typescript
import { MergeGateway } from "merge-gateway-sdk";

const client = new MergeGateway({
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
  baseUrl: process.env.MERGE_GATEWAY_BASE_URL + "/v1",
});

async function main() {
  const response = await client.responses.create({
    model: "openai/gpt-4o",
    input: [
      { type: "message", role: "user", content: "Say 'Gateway integration successful!' and nothing else." },
    ],
  });
  console.log(response.output[0].content[0].text);
}

main();
```

Ask the user if they want to run the test script.

## Cross-Cutting Rules

- **Never delete old configuration** — comment it out with a note about the replacement.
- **Idempotency** — Before making changes, check if `MERGE_GATEWAY_BASE_URL` or `api-gateway.merge.dev` is already present. If so, skip those files and tell the user.
- **Both languages** — Support Python and TypeScript/Node.js patterns. Detect which is in use from the project.
- **Provider-prefixed models** — ALL model names must use the `provider/model` format when going through Gateway.
- **Base URL (Merge Gateway SDK)** — Append `/v1` to the base URL: `MERGE_GATEWAY_BASE_URL + "/v1"`.
- **Base URL (OpenAI SDK alternative)** — Append `/v1/openai` to the base URL: `MERGE_GATEWAY_BASE_URL + "/v1/openai"`.
