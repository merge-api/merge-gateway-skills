# Migrate Direct Provider SDKs

Migrate from calling OpenAI, Anthropic, or Google directly to routing through Merge Gateway.

## What it does

- Finds all direct provider SDK usage (OpenAI, Anthropic, Google Generative AI)
- Updates client constructors to use the Merge Gateway SDK
- Converts Google Generative AI code to Merge Gateway SDK format
- Prefixes all model names with their provider (`gpt-4o` → `openai/gpt-4o`)
- Consolidates multiple provider API keys into a single Gateway key

## Install

```bash
claude install-skill https://github.com/merge-api/merge-gateway-skills
```

## Usage

```
/migrate-direct-sdk
```

Claude will scan for all provider SDKs in your project and migrate them one by one.

## Prerequisites

- A Merge Gateway account with an API key (`mg_...`)
- An existing project using OpenAI, Anthropic, or Google AI SDKs
