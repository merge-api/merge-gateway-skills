# Build a Tool-Use Agent

Scaffold a function-calling agent loop using the OpenAI SDK routed through Merge Gateway.

## What it does

- Gathers your agent requirements (purpose, tools, model preference)
- Detects Python or TypeScript from your project
- Scaffolds a complete agent with tool definitions, implementations, and an agent loop
- Customizes tools and system prompt based on your requirements
- Ensures the OpenAI SDK dependency is installed

## Install

```bash
claude install-skill https://github.com/merge-api/merge-gateway-skills
```

## Usage

```
/build-agent
```

Claude will ask about your agent's purpose and required tools, then generate a ready-to-run agent file.

## Prerequisites

- A Merge Gateway account with an API key (`mg_...`)
- An existing project with Python or TypeScript/Node.js
