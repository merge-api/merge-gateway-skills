---
description: Migrate from OpenAI, Anthropic, or Google SDKs to Merge Gateway. Use when the user wants to replace direct provider SDK calls with Gateway.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

# Migrate Direct Provider SDKs to Merge Gateway

Migrate from calling OpenAI, Anthropic, or Google directly to routing through Merge Gateway.

## Language Support

The Merge Gateway SDK is available in both **Python** and **TypeScript/Node**:

- **Python:** `pip install merge-gateway-sdk`
- **TypeScript/Node:** `npm install merge-gateway-sdk`

Detect the user's stack and show the relevant language.

## Steps

### 1. Search for Direct Provider SDK Usage

Search the project for all direct provider SDK usage:

**OpenAI:**
- Imports: `from openai import OpenAI`, `import OpenAI from "openai"`
- Constructors: `OpenAI(`, `new OpenAI({`
- Env vars: `OPENAI_API_KEY`

**Anthropic:**
- Imports: `from anthropic import Anthropic`, `import Anthropic from "@anthropic-ai/sdk"`
- Constructors: `Anthropic(`, `new Anthropic({`
- Env vars: `ANTHROPIC_API_KEY`

**Google Generative AI:**
- Imports: `import google.generativeai`, `from google import generativeai`, `import { GoogleGenerativeAI }`, `@google/generative-ai`
- Constructors: `genai.GenerativeModel(`, `genai.configure(`, `new GoogleGenerativeAI(`
- Env vars: `GOOGLE_API_KEY`, `GEMINI_API_KEY`

Report all findings to the user before making changes.

### 2. Check for Prior Migration

Before making changes, check if `MERGE_GATEWAY` or `api-gateway.merge.dev` already exists in the project. If so, report which parts are already migrated and skip those.

### 3. Migrate Each Detected Provider

For each provider detected in step 1, apply the corresponding migration below. If multiple providers are detected, apply each subsection in order.

#### 3A. Migrate OpenAI SDK

This is the simplest migration — swap to the Merge Gateway SDK.

Python:
```python
# Before
from openai import OpenAI
client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])

# After
from merge_gateway import MergeGateway
client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
)
```

TypeScript:
```typescript
// Before
import OpenAI from "openai";
const client = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

// After
import { MergeGateway } from "merge-gateway-sdk";
const client = new MergeGateway({
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
  baseUrl: process.env.MERGE_GATEWAY_BASE_URL + "/v1",
});
```

Prefix all model names:
- `gpt-4o` → `openai/gpt-4o`
- `gpt-4o-mini` → `openai/gpt-4o-mini`
- `gpt-4-turbo` → `openai/gpt-4-turbo`
- `gpt-3.5-turbo` → `openai/gpt-3.5-turbo`
- `o1` → `openai/o1`
- `o1-mini` → `openai/o1-mini`
- `o3-mini` → `openai/o3-mini`

#### 3B. Migrate Anthropic SDK

Replace the Anthropic SDK with the Merge Gateway SDK. **Note:** You can alternatively keep the Anthropic SDK and point it at Gateway (without `/v1`), but the Merge Gateway SDK is the primary recommended approach.

Python:
```python
# Before
from anthropic import Anthropic
client = Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])

# After
from merge_gateway import MergeGateway
client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
)
```

TypeScript:
```typescript
// Before
import Anthropic from "@anthropic-ai/sdk";
const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

// After
import { MergeGateway } from "merge-gateway-sdk";
const client = new MergeGateway({
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
  baseUrl: process.env.MERGE_GATEWAY_BASE_URL + "/v1",
});
```

Prefix all model names:
- `claude-sonnet-4-20250514` → `anthropic/claude-sonnet-4-20250514`
- `claude-3-5-haiku-20241022` → `anthropic/claude-3-5-haiku-20241022`
- `claude-3-opus-20240229` → `anthropic/claude-3-opus-20240229`

#### 3C. Migrate Google Generative AI

Google's SDK has a different API shape — this requires switching to the Merge Gateway SDK.

**Explain to the user:** Google's `generativeai` SDK uses a different API format. The migration involves switching to the Merge Gateway SDK, which provides a unified interface to all providers including Google models.

Python:
```python
# Before
import google.generativeai as genai
genai.configure(api_key=os.environ["GOOGLE_API_KEY"])
model = genai.GenerativeModel("gemini-pro")
response = model.generate_content("Hello!")
print(response.text)

# After
from merge_gateway import MergeGateway
import os
client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
)
response = client.responses.create(
    model="google/gemini-2.0-flash",
    input=[{"type": "message", "role": "user", "content": "Hello!"}],
)
print(response.output[0].content[0].text)
```

TypeScript:
```typescript
// Before
import { GoogleGenerativeAI } from "@google/generative-ai";
const genAI = new GoogleGenerativeAI(process.env.GOOGLE_API_KEY!);
const model = genAI.getGenerativeModel({ model: "gemini-pro" });
const result = await model.generateContent("Hello!");
console.log(result.response.text());

// After
import { MergeGateway } from "merge-gateway-sdk";
const client = new MergeGateway({
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
  baseUrl: process.env.MERGE_GATEWAY_BASE_URL + "/v1",
});
const response = await client.responses.create({
  model: "google/gemini-2.0-flash",
  input: [{ type: "message", role: "user", content: "Hello!" }],
});
console.log(response.output[0].content[0].text);
```

Google model name mapping:
- `gemini-pro` → `google/gemini-2.0-flash`
- `gemini-pro-vision` → `google/gemini-2.0-flash` (multimodal by default)
- `gemini-1.5-pro` → `google/gemini-1.5-pro`
- `gemini-1.5-flash` → `google/gemini-1.5-flash`

**Message format translation:**
- `generate_content("text")` → `input=[{"type": "message", "role": "user", "content": "text"}]`
- `chat.send_message("text")` → append to input array and call `client.responses.create()`
- Multi-turn: maintain an `input` array instead of using `model.start_chat()`

After migration, the `google-generativeai` / `@google/generative-ai` dependency can be removed if it's no longer used elsewhere. Install `merge-gateway-sdk` if not already present (Python: `pip install merge-gateway-sdk`, TypeScript: `npm install merge-gateway-sdk`).

**Embeddings migration:**

When migrating OpenAI embeddings, the response format is preserved:

```python
# Before (OpenAI)
response = client.embeddings.create(model="text-embedding-3-small", input="Hello")
embedding = response.data[0].embedding

# After (Merge Gateway)
response = client.embeddings.create(model="openai/text-embedding-3-small", input="Hello")
embedding = response.data[0].embedding  # Same response format
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

### 4. Consolidate Environment Variables

Replace multiple provider keys with a single Gateway key:

```
# Before
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_API_KEY=AI...

# After
MERGE_GATEWAY_API_KEY=mg_your_api_key_here
MERGE_GATEWAY_BASE_URL=https://api-gateway.merge.dev
# OPENAI_API_KEY=sk-...          # Replaced by MERGE_GATEWAY_API_KEY
# ANTHROPIC_API_KEY=sk-ant-...   # Replaced by MERGE_GATEWAY_API_KEY
# GOOGLE_API_KEY=AI...           # Replaced by MERGE_GATEWAY_API_KEY
```

Comment out (do NOT delete) old provider env vars.

### 5. Verify

Generate a test script matching the SDK(s) that were migrated:

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
    input=[{"type": "message", "role": "user", "content": "Say 'Migration successful!' and nothing else."}],
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
    input: [{ type: "message", role: "user", content: "Say 'Migration successful!' and nothing else." }],
  });
  console.log(response.output[0].content[0].text);
}

main();
```

## Cross-Cutting Rules

- **Never delete old configuration** — comment out old env vars with a note about the replacement.
- **Idempotency** — Check if migration is already partially applied before making changes.
- **Provider-prefixed models** — ALL model names must use `provider/model` format.
- **Base URL** — The env var `MERGE_GATEWAY_BASE_URL` should be set **without** `/v1` (e.g., `https://api-gateway.merge.dev`). Always append `/v1` in code. If the env var already contains `/v1`, do NOT append it again — check for this to avoid a double `/v1` path.
- **Anthropic SDK compatibility** — If keeping Anthropic SDK as alternative, do NOT append `/v1`: `os.environ["MERGE_GATEWAY_BASE_URL"]`.

