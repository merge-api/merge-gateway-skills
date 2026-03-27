# Merge Gateway — Claude Code Skills

A Claude Code plugin with skills for integrating with, building on, and migrating to [Merge Gateway](https://api-gateway.merge.dev).

## Installation

Add the plugin marketplace, then install:

```bash
claude plugin marketplace add merge-api/merge-gateway-skills
claude plugin install merge-gateway
```

That's it — the skills are ready to use immediately.

### Alternative: manual install

If you prefer to add skills directly to a single project:

```bash
git clone https://github.com/merge-api/merge-gateway-skills.git
cp -r merge-gateway-skills/.claude/skills/ /path/to/your/project/.claude/skills/
```

## Getting Started

After installing, open Claude Code and try one of these:

### Integrate Gateway into your project
```
/merge-gateway:gateway-implement
```

### Build an agent with tool calling
```
/merge-gateway:build-agent
```

### Migrate from another provider
```
/merge-gateway:migrate-openrouter
/merge-gateway:migrate-direct-sdk
/merge-gateway:migrate-bedrock
/merge-gateway:migrate-azure
/merge-gateway:migrate-litellm
```

Or just describe what you want — Claude will pick the right skill:

- "Add Merge Gateway to this project"
- "Build a tool-use agent that can search the web"
- "Migrate this project from OpenRouter to Merge Gateway"
- "Switch from Bedrock to Merge Gateway"

## Available Skills

### Integration

| Skill | Command | What it does |
|-------|---------|-------------|
| **Gateway Implementation** | `/gateway-implement` | Detect your stack, install the SDK, and verify the integration |
| **Build a Tool-Use Agent** | `/build-agent` | Scaffold an agent with tool definitions and an execution loop |

### Migration

| Skill | Command | What it does |
|-------|---------|-------------|
| **Migrate from OpenRouter** | `/migrate-openrouter` | Swap URLs, headers, and params for Gateway equivalents |
| **Migrate Direct SDKs** | `/migrate-direct-sdk` | Move from OpenAI, Anthropic, or Google SDKs to Gateway |
| **Migrate from Bedrock** | `/migrate-bedrock` | Replace boto3 calls and map Bedrock model IDs |
| **Migrate from Azure OpenAI** | `/migrate-azure` | Map deployment names and remove Azure-specific config |
| **Migrate from LiteLLM** | `/migrate-litellm` | Replace LiteLLM proxy or library with managed Gateway |

## What You'll Need

- A Merge Gateway API key (`mg_...`) — get one from the [Gateway dashboard](https://api-gateway.merge.dev)
- Your Gateway base URL (default: `https://api-gateway.merge.dev`)

## How It Works

All skills route your LLM calls through Merge Gateway using the **Merge Gateway SDK**:

```python
from merge_gateway import MergeGateway

client = MergeGateway(api_key="mg_...")

response = client.responses.create(
    model="openai/gpt-4o",
    input=[{"type": "message", "role": "user", "content": "Hello!"}],
)
print(response.output[0].content[0].text)
```

Key conventions:
- **Model names** use `provider/model` format (e.g., `openai/gpt-4o`, `anthropic/claude-sonnet-4-20250514`)
- **Auth** uses a single `mg_...` API key for all providers
