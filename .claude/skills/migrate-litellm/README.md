# Migrate from LiteLLM

Replace a self-hosted LiteLLM proxy or the LiteLLM Python library with managed Merge Gateway.

## What it does

- Detects whether you're using the LiteLLM proxy, library, or both
- Migrates OpenAI SDK clients pointed at a LiteLLM proxy to Gateway
- Replaces `litellm.completion()` / `litellm.acompletion()` calls with OpenAI SDK
- Removes LiteLLM-specific configuration (callbacks, verbose mode, `drop_params`)
- Consolidates provider API keys into a single Gateway key
- Flags LiteLLM infrastructure (Docker, Kubernetes) for cleanup

## Install

```bash
claude install-skill https://github.com/merge-api/merge-gateway-skills
```

## Usage

```
/migrate-litellm
```

Claude will determine your LiteLLM setup (proxy vs. library) and walk you through the migration.

## Prerequisites

- A Merge Gateway account with an API key (`mg_...`)
- An existing project using LiteLLM (proxy or Python library)
