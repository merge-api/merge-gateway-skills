# Gateway Implementation

Add Merge Gateway to an existing project. This skill detects your stack, installs the Merge Gateway SDK, updates your code to route through Gateway, and verifies the integration works.

## What it does

- Scans your project for existing LLM SDK usage (OpenAI, Anthropic, LangChain, etc.)
- Installs the Merge Gateway SDK (`merge-gateway` for Python, `merge-gateway-sdk` for Node)
- Migrates client constructors and API calls to the Merge Gateway unified format
- Converts model names to the `provider/model` format (e.g., `openai/gpt-4o`)
- Sets up `MERGE_GATEWAY_API_KEY` and `MERGE_GATEWAY_BASE_URL` environment variables
- Generates a test script to verify the integration

## Install

```bash
claude install-skill https://github.com/merge-api/merge-gateway-skills
```

## Usage

```
/gateway-implement
```

Claude will walk you through the integration step by step, starting with stack detection.

## Prerequisites

- A Merge Gateway account with an API key (`mg_...`)
- An existing project with Python or TypeScript/Node.js
