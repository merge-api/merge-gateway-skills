
# Integrate Merge Gateway

Guide a developer through adding Merge Gateway to an existing project. Detect their stack, install the Merge Gateway SDK, update configuration, and verify the integration works.

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
### 1. Detect Stack

Search the project for existing LLM SDK usage:

- **Python:** Check `requirements.txt`, `pyproject.toml`, `Pipfile`, `setup.py` for `openai`, `anthropic`, `langchain`, `litellm`, `boto3`, `google-generativeai`
- **TypeScript/Node:** Check `package.json` for `openai`, `@anthropic-ai/sdk`, `langchain`, `litellm`, `@google/generative-ai`, `@azure/openai`

Search for existing LLM client constructors:
- `OpenAI(`, `new OpenAI({`, `Anthropic(`, `new Anthropic({`, `AzureOpenAI(`, `boto3.client('bedrock`

Report what you found to the user before proceeding.

### 2. Set Up Credentials

Check if `MERGE_GATEWAY_API_KEY` is already set in the environment or `.env` file. If it is, confirm with the user and move on.

If not, walk the user through these steps:

**Step 2a.** Tell the user to go to **https://gateway.merge.dev** and create or copy their API key (it starts with `mg_`).

**Step 2b.** Ask the user: **"Are you setting this up for local development or a deployed environment?"**

- **Local development:** Tell the user to run this command in their terminal, replacing `mg_YOUR_KEY` with their actual API key:
  ```
  ! echo "MERGE_GATEWAY_API_KEY=mg_YOUR_KEY" >> .env
  ```
  Then check that `.gitignore` includes `.env`. If not, add it and warn the user that `.env` files should never be committed to git.

- **Deployed / CI/CD:** Tell the user to add `MERGE_GATEWAY_API_KEY` as an environment variable in their secrets manager (e.g., AWS Secrets Manager, HashiCorp Vault, Vercel/Netlify env vars, or their CI/CD platform's secrets configuration). Do not create a `.env` file.

**IMPORTANT:** Never ask the user to paste their API key into the Claude conversation. Always give them a terminal command or instructions they execute themselves.

### 3. Install the Merge Gateway SDK

Python:
```bash
pip3 install merge-gateway-sdk
```

TypeScript/Node:
```bash
npm install merge-gateway-sdk
```

### 4. Update SDK Configuration

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
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
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

### 5. Detect Centralized Config

Before making scattered changes, search for centralized configuration patterns:
- Python: `config.py`, `settings.py`, `.env` loading, `pydantic.BaseSettings`
- TypeScript: `config.ts`, `settings.ts`, DI containers, `process.env` centralization

If a centralized config exists, update the Gateway settings there and have call sites reference the config. If not, update each call site directly.

### 6. Set Up Environment Variables

Verify that `MERGE_GATEWAY_API_KEY` is configured (either in `.env` for local dev, or in the user's secrets manager for deployed environments — this was done in Step 2).

The SDK automatically connects to `https://api-gateway.merge.dev/v1` — no base URL configuration needed.

If there are old provider API key env vars, comment them out (do NOT delete):
```
# OPENAI_API_KEY=sk-...  # Replaced by MERGE_GATEWAY_API_KEY
```

If the user has a `.env` file, verify `.gitignore` includes `.env`.

### 7. Verify Integration

Generate a test script, write it to the project, and run it. The test must produce a visible successful response.

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
    input=[
        {"type": "message", "role": "user", "content": "What is 2 + 2? Reply with just the number."},
    ],
)
print("Gateway response:", response.output[0].content[0].text)
print("Model used:", response.model)
print("Integration verified successfully!")
```

TypeScript (`test_gateway.ts`):
```typescript
import { MergeGateway } from "merge-gateway-sdk";
import dotenv from "dotenv";

dotenv.config();

const client = new MergeGateway({
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
});

async function main() {
  const response = await client.responses.create({
    model: "openai/gpt-4o-mini",
    input: [
      { type: "message", role: "user", content: "What is 2 + 2? Reply with just the number." },
    ],
  });
  console.log("Gateway response:", response.output[0].content[0].text);
  console.log("Model used:", response.model);
  console.log("Integration verified successfully!");
}

main();
```

**Run the test script** and confirm it prints a response. If it fails, check:
- Is `MERGE_GATEWAY_API_KEY` set correctly in `.env`?
- Is `dotenv` / `python-dotenv` installed for loading `.env` files?

**Embeddings migration:**

The Gateway SDK also supports embeddings. The response format matches the OpenAI SDK:

Python:
```python
# Before (OpenAI)
response = client.embeddings.create(model="text-embedding-3-small", input="Hello")
embedding = response.data[0].embedding

# After (Merge Gateway)
response = client.embeddings.create(model="openai/text-embedding-3-small", input="Hello")
embedding = response.data[0].embedding
```

TypeScript:
```typescript
// Before (OpenAI)
const response = await client.embeddings.create({ model: "text-embedding-3-small", input: "Hello" });
const embedding = response.data[0].embedding;

// After (Merge Gateway)
const response = await client.embeddings.create({ model: "openai/text-embedding-3-small", input: "Hello" });
const embedding = response.data[0].embedding;
```

**Streaming:**

The Gateway SDK supports streaming responses via Server-Sent Events:

Python:
```python
response = client.responses.create(
    model="openai/gpt-4o",
    input=[{"type": "message", "role": "user", "content": "Tell me a story."}],
    stream=True,
)

with response as stream:
    for event in stream:
        if "delta" in event:
            print(event["delta"].get("text", ""), end="", flush=True)
print()
```

TypeScript:
```typescript
const stream = await client.responses.create({
  model: "openai/gpt-4o",
  input: [{ type: "message", role: "user", content: "Tell me a story." }],
  stream: true,
});

for await (const event of stream) {
  const delta = event["delta"] as Record<string, unknown> | undefined;
  if (delta?.text) {
    process.stdout.write(String(delta.text));
  }
}
console.log();
```

> **Note:** Stream events are raw JSON objects (dicts in Python, `Record<string, unknown>` in TypeScript). The event structure follows the SSE format from the API.

Ask the user if they want to run the test script.

### 8. Error Handling

Show the user how to handle common Gateway errors. The SDK provides typed exceptions for each error case:

Python:
```python
from merge_gateway import MergeGateway, AuthenticationError, BadRequestError, RateLimitError, APIError

client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
)

try:
    response = client.responses.create(
        model="openai/gpt-4o",
        input=[{"type": "message", "role": "user", "content": "Hello!"}],
    )
    print(response.output[0].content[0].text)
except AuthenticationError:
    print("Invalid API key. Check MERGE_GATEWAY_API_KEY.")
except BadRequestError as e:
    print(f"Bad request: {e.message}")
except RateLimitError:
    print("Rate limit hit. Back off and retry.")
except APIError as e:
    # Catches all other API errors (including 402 budget exceeded)
    print(f"API error {e.status_code}: {e.message}")
```

TypeScript:
```typescript
import {
  MergeGateway,
  AuthenticationError,
  BadRequestError,
  RateLimitError,
  APIError,
} from "merge-gateway-sdk";

const client = new MergeGateway({
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
});

try {
  const response = await client.responses.create({
    model: "openai/gpt-4o",
    input: [{ type: "message", role: "user", content: "Hello!" }],
  });
  console.log(response.output[0].content[0].text);
} catch (e) {
  if (e instanceof AuthenticationError) {
    console.error("Invalid API key. Check MERGE_GATEWAY_API_KEY.");
  } else if (e instanceof BadRequestError) {
    console.error(`Bad request: ${e.message}`);
  } else if (e instanceof RateLimitError) {
    console.error("Rate limit hit. Back off and retry.");
  } else if (e instanceof APIError) {
    // Catches all other API errors (including 402 budget exceeded)
    console.error(`API error ${e.statusCode}: ${e.message}`);
  }
}
```

**Common error codes:**
| Status | Exception | Meaning |
|---|---|---|
| 400 | `BadRequestError` | Invalid request format or parameters |
| 401 | `AuthenticationError` | Invalid or missing API key |
| 402 | `APIError` (status_code=402) | Free tier budget exhausted — upgrade to Pro |
| 404 | `NotFoundError` | Model or endpoint not found |
| 429 | `RateLimitError` | Too many requests — implement backoff |

All exceptions inherit from `MergeGatewayError` and have `.message`, `.status_code`, and `.body` attributes.

## Cross-Cutting Rules

- **Never delete old configuration** — comment it out with a note about the replacement.
- **Idempotency** — Before making changes, check if `MERGE_GATEWAY_API_KEY` or `MergeGateway` is already present. If so, skip those files and tell the user.
- **Provider-prefixed models** — ALL model names must use the `provider/model` format when going through Gateway.
- **Base URL** — The SDK defaults to `https://api-gateway.merge.dev/v1`. Only pass `base_url`/`baseUrl` if the user has a custom gateway endpoint.
