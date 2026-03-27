# Merge Gateway — Claude Code Skills

A set of Claude Code skills to help you integrate with, build on, and migrate to [Merge Gateway](https://api-gateway.merge.dev).

## Installation

Copy the `.claude/` directory into your project root:

```bash
# From this directory
cp -r .claude/ /path/to/your/project/.claude/

# Or clone and copy
git clone <repo-url> merge-gateway-skills
cp -r merge-gateway-skills/.claude/ /path/to/your/project/.claude/
```

If your project already has a `.claude/` directory, merge the `skills/` folder:

```bash
cp -r .claude/skills/ /path/to/your/project/.claude/skills/
```

## Available Skills

### Integration

| Skill | Description |
|---|---|
| **gateway-implement** | Add Merge Gateway to an existing project. Detects your stack, updates SDK config, sets up env vars, and verifies the integration. |
| **build-agent** | Scaffold a function-calling agent loop using the Merge Gateway SDK. Generates a complete agent with tool definitions, execution loop, and error handling. |

### Migration

| Skill | Description |
|---|---|
| **migrate-openrouter** | Migrate from OpenRouter to Gateway. Swaps URLs, removes OpenRouter-specific headers and params. |
| **migrate-direct-sdk** | Migrate from calling OpenAI/Anthropic/Google directly to routing through Gateway. Handles all three SDKs. |
| **migrate-bedrock** | Migrate from AWS Bedrock (boto3) to the Merge Gateway SDK. Converts request/response formats. |
| **migrate-azure** | Migrate from Azure OpenAI to the Merge Gateway SDK. Maps deployment names to model names. |
| **migrate-litellm** | Migrate from self-hosted LiteLLM proxy or library to Gateway. Includes feature mapping guide. |

## Usage

Once installed, use the skills in Claude Code by describing what you want to do:

- "Integrate Merge Gateway into this project"
- "Build a tool-use agent that can search the web and query my database"
- "Migrate this project from OpenRouter to Merge Gateway"
- "Migrate from Azure OpenAI to Merge Gateway"
- "Switch from Bedrock to Merge Gateway"
- "Replace LiteLLM with Merge Gateway"
- "Migrate from direct OpenAI/Anthropic SDK calls to Gateway"

## What You'll Need

- A Merge Gateway API key (`mg_...`) — get one from the [Gateway dashboard](https://api-gateway.merge.dev)
- Your Gateway base URL (default: `https://api-gateway.merge.dev`)

## How It Works

All skills route your LLM calls through Merge Gateway using the **Merge Gateway SDK**. The pattern is:

```python
from merge_gateway import MergeGateway
import os

client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
)

response = client.responses.create(
    model="openai/gpt-4o",  # Provider-prefixed model name
    input=[{"type": "message", "role": "user", "content": "Hello!"}],
)
```

Alternatively, you can use the **OpenAI SDK** with Gateway's compatibility endpoint:

```python
from openai import OpenAI

client = OpenAI(
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1/openai",
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
)

response = client.chat.completions.create(
    model="openai/gpt-4o",
    messages=[{"role": "user", "content": "Hello!"}],
)
```

Key conventions:
- **Merge Gateway SDK base URL** appends `/v1`
- **OpenAI SDK base URL** appends `/v1/openai`
- **Anthropic SDK base URL** does NOT include `/v1`
- **Model names** use `provider/model` format (e.g., `openai/gpt-4o`, `anthropic/claude-sonnet-4-20250514`)
- **Auth** uses a single `mg_...` API key for all providers
