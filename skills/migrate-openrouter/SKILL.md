---
description: Migrate from OpenRouter to Merge Gateway. Use when the user wants to switch from OpenRouter, replace OpenRouter, or move off OpenRouter.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

# Migrate from OpenRouter to Merge Gateway

Find and replace all OpenRouter references with Merge Gateway equivalents.

## Language Support

The Merge Gateway SDK is available in both **Python** and **TypeScript/Node**:

- **Python:** `pip3 install merge-gateway-sdk`
- **TypeScript/Node:** `npm install merge-gateway-sdk`

Detect the user's stack and show the relevant language.

## Steps

### 0. Check for Plugin Updates

Before proceeding, ensure the user has the latest version of the Merge Gateway skills by running:
```
claude plugin update merge-gateway
```
### 1. Search for OpenRouter Usage

Search the entire project for OpenRouter references:

- **URLs:** `openrouter.ai` in any file (especially `openrouter.ai/api/v1`)
- **Env vars:** `OPENROUTER_API_KEY`, `OPENROUTER_BASE_URL`, `OPENROUTER_API_BASE`
- **Headers:** `HTTP-Referer`, `X-Title` (OpenRouter-specific headers)
- **Config files:** `.env`, `.env.example`, `.env.local`, `docker-compose.yml`, `config.yaml`
- **Package imports:** Check if `openai` SDK is already installed (OpenRouter uses the OpenAI SDK)

Report all findings to the user before making changes.

### 2. Check for Prior Migration

Before making changes, check if `MERGE_GATEWAY` or `api-gateway.merge.dev` already exists in the project. If so, report which parts are already migrated and skip those.

### 3. Migrate URLs

Replace OpenRouter clients with the Merge Gateway SDK. The SDK handles the base URL automatically — no URL configuration needed.

Python:
```python
# Before
client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key=os.environ["OPENROUTER_API_KEY"],
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
const client = new OpenAI({
  baseURL: "https://openrouter.ai/api/v1",
  apiKey: process.env.OPENROUTER_API_KEY,
});

// After
import { MergeGateway } from "merge-gateway-sdk";
const client = new MergeGateway({
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
});
```

### 4. Migrate Model Names

OpenRouter uses `provider/model` format — the same format Gateway uses. Most model names transfer directly:

- `openai/gpt-4o` → `openai/gpt-4o` (no change)
- `anthropic/claude-3.5-sonnet` → `anthropic/claude-sonnet-4-20250514` (update to latest)
- `google/gemini-pro` → `google/gemini-2.0-flash` (update to latest)
- `meta-llama/llama-3.1-70b-instruct` → `meta-llama/llama-3.1-70b-instruct` (check availability)

Ask the user to confirm model mappings, especially for less common models. Some OpenRouter models may not be available on Gateway.

### 5. Remove OpenRouter-Specific Parameters

Search for and remove these OpenRouter-specific parameters from API calls:

- `transforms` — OpenRouter prompt transforms; Gateway doesn't use these
- `route` — OpenRouter routing parameter (e.g., `"fallback"`); Gateway handles routing via dashboard policies
- `provider` — OpenRouter provider preferences; Gateway manages this via routing policies
- `models` — OpenRouter multi-model fallback array; Gateway handles fallbacks in routing policies

```python
# Before
response = client.chat.completions.create(
    model="openai/gpt-4o",
    messages=messages,
    transforms=["middle-out"],
    route="fallback",
    extra_body={"provider": {"order": ["OpenAI", "Azure"]}},
)

# After
response = client.responses.create(
    model="openai/gpt-4o",
    input=[{"type": "message", "role": msg["role"], "content": msg["content"]} for msg in messages],
)
```

### 6. Remove OpenRouter-Specific Headers

Search for and remove:
- `HTTP-Referer` header (OpenRouter uses this for app identification)
- `X-Title` header (OpenRouter uses this for app naming on their dashboard)

```python
# Before
client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key=os.environ["OPENROUTER_API_KEY"],
    default_headers={
        "HTTP-Referer": "https://myapp.com",
        "X-Title": "My App",
    },
)

# After
from merge_gateway import MergeGateway
client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
)
```

If `default_headers` contained other headers besides OpenRouter-specific ones, keep those.

### 7. Update Environment Variables

Old OpenRouter keys are replaced by a single `MERGE_GATEWAY_API_KEY`.

**First, ask the user:** "Are you setting this up for **local development** or a **deployed environment**?"

- **Local development:** Tell the user to run this in their terminal, replacing `mg_YOUR_KEY` with their actual key from [gateway.merge.dev](https://gateway.merge.dev):
  ```
  ! echo "MERGE_GATEWAY_API_KEY=mg_YOUR_KEY" >> .env
  ```
  Then verify `.gitignore` includes `.env`. Comment out (do NOT delete) old OpenRouter keys in `.env`:
  ```
  # OPENROUTER_API_KEY=...            # Replaced by MERGE_GATEWAY_API_KEY
  ```

- **Deployed / CI/CD:** Tell the user to add `MERGE_GATEWAY_API_KEY` to their secrets manager or CI/CD environment variables, and remove the old OpenRouter key (`OPENROUTER_API_KEY`) from their secrets configuration.

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

## Feature Mapping

Explain to the user how OpenRouter features map to Gateway:

| OpenRouter | Merge Gateway |
|---|---|
| `route: "fallback"` | Routing policies with fallback strategies (configured in dashboard) |
| `transforms` | Not needed — Gateway handles prompt formatting |
| `provider.order` | Routing policy provider priorities (configured in dashboard) |
| Usage dashboard on openrouter.ai | Gateway usage dashboard |
| Per-model pricing | Unified billing through Gateway |

## Cross-Cutting Rules

- **Never delete old configuration** — comment out old env vars with a note about the replacement.
- **Idempotency** — Check if migration is already partially applied before making changes.
- **Provider-prefixed models** — ALL model names must use `provider/model` format.
- **Base URL** — The SDK defaults to `https://api-gateway.merge.dev/v1`. Only pass `base_url`/`baseUrl` if the user has a custom gateway endpoint.
