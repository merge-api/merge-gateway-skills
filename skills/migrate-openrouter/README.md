# Migrate from OpenRouter

Swap OpenRouter URLs, headers, and params for Merge Gateway equivalents.

## What it does

- Finds all OpenRouter references (URLs, env vars, headers)
- Replaces `openrouter.ai/api/v1` base URLs with Gateway
- Removes OpenRouter-specific headers (`HTTP-Referer`, `X-Title`)
- Removes OpenRouter-specific parameters (`transforms`, `route`, `provider`)
- Updates model names where needed (most transfer directly)
- Migrates environment variables

## Install

```bash
claude plugin marketplace add merge-api/merge-gateway-skills
claude plugin install merge-gateway
```

## Usage

```
/migrate-openrouter
```

Claude will scan your project for OpenRouter usage and walk you through each replacement.

## Prerequisites

- A Merge Gateway account with an API key (`mg_...`)
- An existing project using OpenRouter
