---
description: Migrate from OpenAI, Anthropic, or Google SDKs to Merge Gateway. Use when the user wants to replace direct provider SDK calls with Gateway.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

# Migrate Direct Provider SDKs to Merge Gateway

Migrate from calling OpenAI, Anthropic, or Google directly to routing through Merge Gateway.

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

**Choose your migration path:** Ask the user whether they want the quick migration (Option A) or the full migration (Option B).

**STOP here and wait for the user's response.** Do NOT proceed until they answer. Once they answer, follow ONLY the matching option below:

**Option A — Quick migration (keep OpenAI SDK):** Change two lines and all your existing `chat.completions.create()` calls work through Gateway immediately. No API call changes needed.

Python:
```python
# Just change the import config — all chat.completions.create() calls work unchanged
from openai import OpenAI
client = OpenAI(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url="https://api-gateway.merge.dev/v1/openai",
)
# Model names need provider prefix: "gpt-4o" → "openai/gpt-4o"
```

TypeScript:
```typescript
// Just change the config — all chat.completions.create() calls work unchanged
import OpenAI from "openai";
const client = new OpenAI({
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
  baseURL: "https://api-gateway.merge.dev/v1/openai",
});
// Model names need provider prefix: "gpt-4o" → "openai/gpt-4o"
```

**Option B — Full migration (native Merge Gateway SDK):** Switch to the Merge Gateway SDK for full access to tags, routing metadata, multi-model routing, and model discovery.

Python:
```python
# Before
from openai import OpenAI
client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])

# After
from merge_gateway import MergeGateway
client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
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
});
```

Prefix all model names:
- `claude-sonnet-4-6-20250514` → `anthropic/claude-sonnet-4-6-20250514`
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

After migration, the `google-generativeai` / `@google/generative-ai` dependency can be removed if it's no longer used elsewhere. Install `merge-gateway-sdk` if not already present (Python: `pip3 install merge-gateway-sdk`, TypeScript: `npm install merge-gateway-sdk`).

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

Multiple provider API keys are replaced by a single `MERGE_GATEWAY_API_KEY`.

**First, ask the user:** "Are you setting this up for **local development** or a **deployed environment**?"

**STOP here and wait for the user's response.** Do NOT proceed until they answer. Once they answer, follow ONLY the matching path below:

- **Local development:** Tell the user to run this in their terminal, replacing `mg_YOUR_KEY` with their actual key from [gateway.merge.dev](https://gateway.merge.dev):
  ```
  ! echo "MERGE_GATEWAY_API_KEY=mg_YOUR_KEY" >> .env
  ```
  Then verify `.gitignore` includes `.env`. Comment out (do NOT delete) old provider keys in `.env`:
  ```
  # OPENAI_API_KEY=sk-...          # Replaced by MERGE_GATEWAY_API_KEY
  # ANTHROPIC_API_KEY=sk-ant-...   # Replaced by MERGE_GATEWAY_API_KEY
  # GOOGLE_API_KEY=AI...           # Replaced by MERGE_GATEWAY_API_KEY
  ```

- **Deployed / CI/CD:** Tell the user to add `MERGE_GATEWAY_API_KEY` to their secrets manager or CI/CD environment variables, and remove the old provider keys (`OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GOOGLE_API_KEY`) from their secrets configuration.

**Never** ask the user to paste their API key into the Claude conversation. Always give them a command or instructions they execute themselves.

### 5. Verify

Generate a test script matching the SDK(s) that were migrated:

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
- **Idempotency** — Check if migration is already partially applied before making changes.
- **Provider-prefixed models** — ALL model names must use `provider/model` format.
- **Base URL** — The SDK defaults to `https://api-gateway.merge.dev/v1`. Only pass `base_url`/`baseUrl` if the user has a custom gateway endpoint.
- **Anthropic SDK compatibility** — If keeping Anthropic SDK as alternative, pass `base_url="https://api-gateway.merge.dev"` (without `/v1`, since the Anthropic SDK appends its own path).

