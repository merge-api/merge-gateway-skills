
# Migrate Vercel AI SDK to Merge Gateway

Migrate from `@ai-sdk/openai`, `@ai-sdk/anthropic`, `@ai-sdk/google`, or `@openrouter/ai-sdk-provider` to the native Merge Gateway AI SDK provider (`merge-gateway-ai-sdk-provider`).

The Vercel AI SDK framework (`generateText`, `streamText`, `embedMany`) stays the same — only the provider changes.

## Steps

### 0. Check for Plugin Updates

Before proceeding, ensure the user has the latest version of the Merge Gateway skills by running:
```
claude plugin update merge-gateway@merge-gateway-skills
```

**Run this update and wait for it to complete before continuing to Step 1.**

### 1. Search for AI SDK Provider Usage

Search the project for all Vercel AI SDK provider usage:

**Provider imports:**
- `@ai-sdk/openai`: `createOpenAI`, `openai`
- `@ai-sdk/anthropic`: `createAnthropic`, `anthropic`
- `@ai-sdk/google`: `createGoogleGenerativeAI`, `google`
- `@openrouter/ai-sdk-provider`: `createOpenRouter`, `openrouter`
- `@ai-sdk/openai-compatible`: `createOpenAICompatible`

**Framework functions:**
- `generateText(`, `streamText(`, `generateObject(`, `streamObject(`
- `embedMany(`, `embed(`
- `import { ... } from "ai"`

**Environment variables:**
- `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GOOGLE_API_KEY`, `GOOGLE_GENERATIVE_AI_API_KEY`, `OPENROUTER_API_KEY`

Report all findings to the user before making changes.

### 2. Check for Prior Migration

Before making changes, check if `merge-gateway-ai-sdk-provider`, `createMergeGateway`, or `MERGE_GATEWAY_API_KEY` already exists in the project. If so, report which parts are already migrated and skip those.

### 3. Choose Migration Path

Ask the user: **"Do you want the native Merge Gateway provider (recommended) or the quick URL shim?"**

**STOP here and wait for the user's response.** Do NOT proceed until they answer. Once they answer, follow ONLY the matching option below.

#### Option A — URL shim (keep existing AI SDK provider, swap config only)

This is the fastest migration — change only the provider configuration. No new packages needed.

```typescript
// Before — @ai-sdk/openai pointing at OpenAI directly
import { createOpenAI } from "@ai-sdk/openai";
const openai = createOpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

// After — same package, pointed at Gateway
const gateway = createOpenAI({
  apiKey: process.env.MERGE_GATEWAY_API_KEY,
  baseURL: "https://api-gateway.merge.dev/v1/ai-sdk",
});

// Model names need provider prefix: "gpt-4o" → "openai/gpt-4o"
```

For `@ai-sdk/anthropic`:
```typescript
// Before
import { createAnthropic } from "@ai-sdk/anthropic";
const anthropic = createAnthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

// After — switch to @ai-sdk/openai pointed at Gateway (OpenAI-compatible)
import { createOpenAI } from "@ai-sdk/openai";
const gateway = createOpenAI({
  apiKey: process.env.MERGE_GATEWAY_API_KEY,
  baseURL: "https://api-gateway.merge.dev/v1/ai-sdk",
});
// model: anthropic("claude-sonnet-4-20250514") → model: gateway("anthropic/claude-sonnet-4-20250514")
```

Install `@ai-sdk/openai` if not already present: `npm install @ai-sdk/openai`

#### Option B — Native provider (recommended)

Install the Merge Gateway provider:
```bash
npm install merge-gateway-ai-sdk-provider
```

Then replace each provider:

**From `@ai-sdk/openai`:**
```typescript
// Before
import { createOpenAI } from "@ai-sdk/openai";
const openai = createOpenAI({ apiKey: process.env.OPENAI_API_KEY });
const { text } = await generateText({ model: openai("gpt-4o"), prompt: "Hello" });

// After
import { createMergeGateway } from "merge-gateway-ai-sdk-provider";
const gateway = createMergeGateway({ apiKey: process.env.MERGE_GATEWAY_API_KEY });
const { text } = await generateText({ model: gateway("openai/gpt-4o"), prompt: "Hello" });
```

**From `@ai-sdk/anthropic`:**
```typescript
// Before
import { createAnthropic } from "@ai-sdk/anthropic";
const anthropic = createAnthropic({ apiKey: process.env.ANTHROPIC_API_KEY });
const { text } = await generateText({ model: anthropic("claude-sonnet-4-20250514"), prompt: "Hello" });

// After
import { createMergeGateway } from "merge-gateway-ai-sdk-provider";
const gateway = createMergeGateway({ apiKey: process.env.MERGE_GATEWAY_API_KEY });
const { text } = await generateText({ model: gateway("anthropic/claude-sonnet-4-20250514"), prompt: "Hello" });
```

**From `@ai-sdk/google`:**
```typescript
// Before
import { createGoogleGenerativeAI } from "@ai-sdk/google";
const google = createGoogleGenerativeAI({ apiKey: process.env.GOOGLE_API_KEY });
const { text } = await generateText({ model: google("gemini-2.0-flash"), prompt: "Hello" });

// After
import { createMergeGateway } from "merge-gateway-ai-sdk-provider";
const gateway = createMergeGateway({ apiKey: process.env.MERGE_GATEWAY_API_KEY });
const { text } = await generateText({ model: gateway("google/gemini-2.0-flash"), prompt: "Hello" });
```

**From `@openrouter/ai-sdk-provider`:**
```typescript
// Before
import { createOpenRouter } from "@openrouter/ai-sdk-provider";
const openrouter = createOpenRouter({ apiKey: process.env.OPENROUTER_API_KEY });
const { text } = await generateText({ model: openrouter("openai/gpt-4o"), prompt: "Hello" });

// After
import { createMergeGateway } from "merge-gateway-ai-sdk-provider";
const gateway = createMergeGateway({ apiKey: process.env.MERGE_GATEWAY_API_KEY });
const { text } = await generateText({ model: gateway("openai/gpt-4o"), prompt: "Hello" });
// Model names transfer directly — OpenRouter and Gateway use the same provider/model format
```

**Centralized config pattern:** If the project has a shared AI client file (e.g., `ai-client.ts`, `lib/ai.ts`, `config/llm.ts`) that exports the provider instance, update that file and then trace all import sites. When the export name changes (e.g., `export const openai` → `export const gateway`), update every file that imports it:

```typescript
// ai-client.ts — BEFORE
import { createOpenAI } from "@ai-sdk/openai";
export const openai = createOpenAI({ apiKey: process.env.OPENAI_API_KEY });

// ai-client.ts — AFTER
import { createMergeGateway } from "merge-gateway-ai-sdk-provider";
export const gateway = createMergeGateway({ apiKey: process.env.MERGE_GATEWAY_API_KEY });
```

Then in every file that imports it:
```typescript
// BEFORE
import { openai } from "./ai-client.js";
const { text } = await generateText({ model: openai("gpt-4o"), ... });

// AFTER
import { gateway } from "./ai-client.js";
const { text } = await generateText({ model: gateway("openai/gpt-4o"), ... });
```

Search for all import sites with: `grep -r "from.*ai-client" src/` (adjust the filename to match the project's actual config file).

After migration, the old provider packages can be removed if no longer used elsewhere:
```bash
npm uninstall @ai-sdk/openai @ai-sdk/anthropic @ai-sdk/google @openrouter/ai-sdk-provider
```

### 4. Update Model Names

Model names need the `provider/model` prefix format:

**OpenAI models:**
- `gpt-4o` → `openai/gpt-4o`
- `gpt-4o-mini` → `openai/gpt-4o-mini`
- `o1` → `openai/o1`
- `o3-mini` → `openai/o3-mini`

**Anthropic models:**
- `claude-sonnet-4-20250514` → `anthropic/claude-sonnet-4-20250514`
- `claude-3-5-haiku-20241022` → `anthropic/claude-3-5-haiku-20241022`

**Google models:**
- `gemini-2.0-flash` → `google/gemini-2.0-flash`
- `gemini-1.5-pro` → `google/gemini-1.5-pro`

**Note:** If the project already uses `provider/model` format (common with OpenRouter), no model name changes are needed.

### 5. Update Embeddings

If the project uses embeddings, update the embedding model. The method name changes from `.embedding()` to `.textEmbeddingModel()`, and the model name needs a provider prefix:

```typescript
// Before — using @ai-sdk/openai default instance
import { openai } from "@ai-sdk/openai";
const { embeddings } = await embedMany({
  model: openai.embedding("text-embedding-3-small"),
  values: ["hello", "world"],
});

// Before — using createOpenAI() factory
const openai = createOpenAI({ apiKey: process.env.OPENAI_API_KEY });
const { embeddings } = await embedMany({
  model: openai.embedding("text-embedding-3-small"),
  values: ["hello", "world"],
});

// After — both cases become:
const { embeddings } = await embedMany({
  model: gateway.textEmbeddingModel("openai/text-embedding-3-small"),
  values: ["hello", "world"],
});
```

**Key changes:** `.embedding()` → `.textEmbeddingModel()`, and add `openai/` prefix to the model name.

### 6. Consolidate Environment Variables

Multiple provider API keys are replaced by a single `MERGE_GATEWAY_API_KEY`.

**First, ask the user:** "Are you setting this up for **local development** or a **deployed environment**?"

**STOP here and wait for the user's response.** Do NOT proceed until they answer. Once they answer, follow ONLY the matching path below:

- **Local development:** Tell the user to run this in their terminal, replacing `mg_YOUR_KEY` with their actual key from [gateway.merge.dev](https://gateway.merge.dev):
  ```
  ! echo "MERGE_GATEWAY_API_KEY=mg_YOUR_KEY" >> .env
  ```

  **STOP here and wait for the user to confirm they've added their key.** Do NOT proceed until they confirm. Once they confirm, verify `.gitignore` includes `.env`, then comment out (do NOT delete) old provider keys in `.env`:
  ```
  # OPENAI_API_KEY=sk-...              # Replaced by MERGE_GATEWAY_API_KEY
  # ANTHROPIC_API_KEY=sk-ant-...       # Replaced by MERGE_GATEWAY_API_KEY
  # GOOGLE_API_KEY=AI...               # Replaced by MERGE_GATEWAY_API_KEY
  # OPENROUTER_API_KEY=sk-or-...       # Replaced by MERGE_GATEWAY_API_KEY
  ```

- **Deployed / CI/CD:** Tell the user to add `MERGE_GATEWAY_API_KEY` to their secrets manager or CI/CD environment variables, and remove the old provider keys from their secrets configuration. **STOP and wait for the user to confirm before proceeding.**

**Never** ask the user to paste their API key into the Claude conversation. Always give them a command or instructions they execute themselves.

### 7. Show Gateway Features (Optional)

After migration, explain that the native provider supports Gateway-specific features via `providerOptions.mergeGateway`:

```typescript
const { text, providerMetadata } = await generateText({
  model: gateway("openai/gpt-4o"),
  prompt: "Hello!",
  providerOptions: {
    mergeGateway: {
      tags: [{ key: "env", value: "prod" }],
      vendor: "anthropic",
      includeRoutingMetadata: true,
    },
  },
});

// Access routing metadata
const routing = providerMetadata?.mergeGateway?.routing;
console.log(routing?.model_used, routing?.cost_usd);
```

Only show this if the user expressed interest in Gateway features, or after the basic migration is verified.

### 8. Verify

Generate a test script that matches the project's language (JS or TS) and existing dotenv pattern. Write it to the project and run it.

**JavaScript (`test_gateway.js`):**
```javascript
import "dotenv/config";
import { createMergeGateway } from "merge-gateway-ai-sdk-provider";
import { generateText } from "ai";

const gateway = createMergeGateway({
  apiKey: process.env.MERGE_GATEWAY_API_KEY,
});

async function main() {
  const { text } = await generateText({
    model: gateway("openai/gpt-4o-mini"),
    prompt: "What is 2 + 2? Reply with just the number.",
  });
  console.log("Gateway response:", text);
  console.log("Migration verified successfully!");
}

main();
```

**TypeScript (`test_gateway.ts`):**
```typescript
import "dotenv/config";
import { createMergeGateway } from "merge-gateway-ai-sdk-provider";
import { generateText } from "ai";

const gateway = createMergeGateway({
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
});

async function main() {
  const { text } = await generateText({
    model: gateway("openai/gpt-4o-mini"),
    prompt: "What is 2 + 2? Reply with just the number.",
  });
  console.log("Gateway response:", text);
  console.log("Migration verified successfully!");
}

main();
```

Match the project's existing dotenv import style. If the project uses `import "dotenv/config"`, use that. If it uses `dotenv.config()`, use that. Check `package.json` for `"type": "module"` to determine ESM vs CJS imports.

## Cross-Cutting Rules

- **Never delete old configuration** — comment out old env vars with a note about the replacement.
- **Idempotency** — Check if migration is already partially applied before making changes.
- **Provider-prefixed models** — ALL model names must use `provider/model` format.
- **The `ai` package stays** — Only the provider changes. `generateText`, `streamText`, `embedMany`, `generateObject`, `streamObject` all work the same.
- **Streaming works unchanged** — `streamText` with the new provider produces the same stream format.
- **Tool calling works unchanged** — Tools defined with `tool()` and `z.object()` work identically through Gateway.
