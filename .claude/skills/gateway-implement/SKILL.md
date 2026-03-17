# Integrate Merge Gateway

Guide a developer through adding Merge Gateway to an existing project. Detect their stack, update SDK configuration, and verify the integration works.

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
- **Gateway base URL** — default: `https://gateway.merge.dev`

### 3. Update SDK Configuration

**If OpenAI SDK is already present (most common path):**

Find all `OpenAI(` / `new OpenAI({` constructors and update them:

Python:
```python
from openai import OpenAI

client = OpenAI(
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
)
```

TypeScript:
```typescript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: process.env.MERGE_GATEWAY_BASE_URL + "/v1",
  apiKey: process.env.MERGE_GATEWAY_API_KEY,
});
```

Update model names to provider-prefixed format:
- `gpt-4o` → `openai/gpt-4o`
- `gpt-4o-mini` → `openai/gpt-4o-mini`
- `gpt-4-turbo` → `openai/gpt-4-turbo`
- `gpt-3.5-turbo` → `openai/gpt-3.5-turbo`
- `claude-sonnet-4-20250514` → `anthropic/claude-sonnet-4-20250514`
- `claude-3-5-haiku-20241022` → `anthropic/claude-3-5-haiku-20241022`

**If Anthropic SDK is present:**

Find all `Anthropic(` / `new Anthropic({` constructors and update them:

Python:
```python
from anthropic import Anthropic

client = Anthropic(
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"],
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
)
```

TypeScript:
```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({
  baseURL: process.env.MERGE_GATEWAY_BASE_URL,
  apiKey: process.env.MERGE_GATEWAY_API_KEY,
});
```

**IMPORTANT:** For OpenAI SDK, the base URL ends with `/v1`. For Anthropic SDK, it does NOT include `/v1`.

**If no LLM SDK is present:** Install the OpenAI SDK (`pip install openai` or `npm install openai`) and create a client wrapper module.

### 4. Detect Centralized Config

Before making scattered changes, search for centralized configuration patterns:
- Python: `config.py`, `settings.py`, `.env` loading, `pydantic.BaseSettings`
- TypeScript: `config.ts`, `settings.ts`, DI containers, `process.env` centralization

If a centralized config exists, update the Gateway settings there and have call sites reference the config. If not, update each call site directly.

### 5. Set Up Environment Variables

Create or update `.env` (and `.env.example` if it exists):
```
MERGE_GATEWAY_API_KEY=mg_your_api_key_here
MERGE_GATEWAY_BASE_URL=https://gateway.merge.dev
```

Check `.gitignore` includes `.env`. If not, warn the user.

Comment out (do NOT delete) any old provider API key env vars:
```
# OPENAI_API_KEY=sk-...  # Replaced by MERGE_GATEWAY_API_KEY
```

### 6. Verify Integration

Generate a test script that makes one call through Gateway:

Python (`test_gateway.py`):
```python
import os
from openai import OpenAI

client = OpenAI(
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
)

response = client.chat.completions.create(
    model="openai/gpt-4o",
    messages=[{"role": "user", "content": "Say 'Gateway integration successful!' and nothing else."}],
)
print(response.choices[0].message.content)
```

TypeScript (`test_gateway.ts`):
```typescript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: process.env.MERGE_GATEWAY_BASE_URL + "/v1",
  apiKey: process.env.MERGE_GATEWAY_API_KEY,
});

async function main() {
  const response = await client.chat.completions.create({
    model: "openai/gpt-4o",
    messages: [{ role: "user", content: "Say 'Gateway integration successful!' and nothing else." }],
  });
  console.log(response.choices[0].message.content);
}

main();
```

Ask the user if they want to run the test script.

## Cross-Cutting Rules

- **Never delete old configuration** — comment it out with a note about the replacement.
- **Idempotency** — Before making changes, check if `MERGE_GATEWAY_BASE_URL` or `gateway.merge.dev` is already present. If so, skip those files and tell the user.
- **Both languages** — Support Python and TypeScript/Node.js patterns. Detect which is in use from the project.
- **Provider-prefixed models** — ALL model names must use the `provider/model` format when going through Gateway.
