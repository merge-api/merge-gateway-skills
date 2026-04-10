# Migrate from Vercel AI SDK

Switch from `@ai-sdk/openai`, `@ai-sdk/anthropic`, `@ai-sdk/google`, or `@openrouter/ai-sdk-provider` to the native Merge Gateway AI SDK provider.

## What it does

- Finds all AI SDK provider imports and `generateText`/`streamText`/`embedMany` usage
- Offers two migration paths: native provider (recommended) or URL shim (zero-install)
- Replaces provider initialization (`createOpenAI` → `createMergeGateway`)
- Adds `provider/model` prefixes to model names
- Migrates environment variables to `MERGE_GATEWAY_API_KEY`
- Shows optional Gateway features (tags, vendor routing, routing metadata)

## Install

```bash
claude plugin marketplace add merge-api/merge-gateway-skills
claude plugin install merge-gateway
```

## Usage

```
/migrate-ai-sdk
```

Claude will scan your project for AI SDK provider usage and walk you through the migration.

## Prerequisites

- A Merge Gateway account with an API key (`mg_...`)
- An existing project using the Vercel AI SDK (`ai` package)
