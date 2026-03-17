# Migrate Direct Provider SDKs to Merge Gateway

Migrate from calling OpenAI, Anthropic, or Google directly to routing through Merge Gateway.

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

Before making changes, check if `MERGE_GATEWAY` or `gateway.merge.dev` already exists in the project. If so, report which parts are already migrated and skip those.

### 3. Migrate OpenAI SDK (Simplest Path)

This is the simplest migration — just add `base_url` and swap the API key.

Python:
```python
# Before
from openai import OpenAI
client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])

# After
from openai import OpenAI
client = OpenAI(
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
)
```

TypeScript:
```typescript
// Before
import OpenAI from "openai";
const client = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

// After
import OpenAI from "openai";
const client = new OpenAI({
  baseURL: process.env.MERGE_GATEWAY_BASE_URL + "/v1",
  apiKey: process.env.MERGE_GATEWAY_API_KEY,
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

### 4. Migrate Anthropic SDK

Update the base URL and API key. **Note:** Anthropic SDK base URL does NOT include `/v1`.

Python:
```python
# Before
from anthropic import Anthropic
client = Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])

# After
from anthropic import Anthropic
client = Anthropic(
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"],
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
)
```

TypeScript:
```typescript
// Before
import Anthropic from "@anthropic-ai/sdk";
const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

// After
import Anthropic from "@anthropic-ai/sdk";
const client = new Anthropic({
  baseURL: process.env.MERGE_GATEWAY_BASE_URL,
  apiKey: process.env.MERGE_GATEWAY_API_KEY,
});
```

Prefix all model names:
- `claude-sonnet-4-20250514` → `anthropic/claude-sonnet-4-20250514`
- `claude-3-5-haiku-20241022` → `anthropic/claude-3-5-haiku-20241022`
- `claude-3-opus-20240229` → `anthropic/claude-3-opus-20240229`

### 5. Migrate Google Generative AI (Bigger Refactor)

Google's SDK has a different API shape — this requires switching to the OpenAI SDK.

**Explain to the user:** Google's `generativeai` SDK uses a different API format than OpenAI's. The migration involves switching to the OpenAI SDK pointed at Gateway, which provides a unified interface to all providers including Google models.

Python:
```python
# Before
import google.generativeai as genai
genai.configure(api_key=os.environ["GOOGLE_API_KEY"])
model = genai.GenerativeModel("gemini-pro")
response = model.generate_content("Hello!")
print(response.text)

# After
from openai import OpenAI
client = OpenAI(
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
)
response = client.chat.completions.create(
    model="google/gemini-2.0-flash",
    messages=[{"role": "user", "content": "Hello!"}],
)
print(response.choices[0].message.content)
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
import OpenAI from "openai";
const client = new OpenAI({
  baseURL: process.env.MERGE_GATEWAY_BASE_URL + "/v1",
  apiKey: process.env.MERGE_GATEWAY_API_KEY,
});
const response = await client.chat.completions.create({
  model: "google/gemini-2.0-flash",
  messages: [{ role: "user", content: "Hello!" }],
});
console.log(response.choices[0].message.content);
```

Google model name mapping:
- `gemini-pro` → `google/gemini-2.0-flash`
- `gemini-pro-vision` → `google/gemini-2.0-flash` (multimodal by default)
- `gemini-1.5-pro` → `google/gemini-1.5-pro`
- `gemini-1.5-flash` → `google/gemini-1.5-flash`

**Message format translation:**
- `generate_content("text")` → `messages=[{"role": "user", "content": "text"}]`
- `chat.send_message("text")` → append to messages array and call `chat.completions.create()`
- Multi-turn: maintain a `messages` array instead of using `model.start_chat()`

After migration, the `google-generativeai` / `@google/generative-ai` dependency can be removed if it's no longer used elsewhere.

### 6. Consolidate Environment Variables

Replace multiple provider keys with a single Gateway key:

```
# Before
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_API_KEY=AI...

# After
MERGE_GATEWAY_API_KEY=mg_your_api_key_here
MERGE_GATEWAY_BASE_URL=https://gateway.merge.dev
# OPENAI_API_KEY=sk-...          # Replaced by MERGE_GATEWAY_API_KEY
# ANTHROPIC_API_KEY=sk-ant-...   # Replaced by MERGE_GATEWAY_API_KEY
# GOOGLE_API_KEY=AI...           # Replaced by MERGE_GATEWAY_API_KEY
```

Comment out (do NOT delete) old provider env vars.

### 7. Verify

Generate a test script matching the SDK(s) that were migrated:

```python
import os
from openai import OpenAI

client = OpenAI(
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
)

response = client.chat.completions.create(
    model="openai/gpt-4o",
    messages=[{"role": "user", "content": "Say 'Migration successful!' and nothing else."}],
)
print(response.choices[0].message.content)
```

## Cross-Cutting Rules

- **Never delete old configuration** — comment out old env vars with a note about the replacement.
- **Idempotency** — Check if migration is already partially applied before making changes.
- **Provider-prefixed models** — ALL model names must use `provider/model` format.
- **OpenAI SDK base URL** — Append `/v1`: `os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1"`.
- **Anthropic SDK base URL** — Do NOT append `/v1`: `os.environ["MERGE_GATEWAY_BASE_URL"]`.
- **Both languages** — Support Python and TypeScript/Node.js patterns.
