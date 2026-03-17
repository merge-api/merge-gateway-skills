# Gateway Implementation

Add Merge Gateway to an existing project. This skill detects your stack, updates SDK configuration, sets up environment variables, and verifies the integration works.

## What it does

- Scans your project for existing LLM SDK usage (OpenAI, Anthropic, LangChain, etc.)
- Updates SDK client constructors to point at Merge Gateway
- Converts model names to the `provider/model` format
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
