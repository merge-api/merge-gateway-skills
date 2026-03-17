# Migrate from AWS Bedrock

Replace boto3 Bedrock calls with the OpenAI SDK through Merge Gateway.

## What it does

- Finds all AWS Bedrock usage (`invoke_model`, `converse`, streaming calls)
- Maps Bedrock model IDs to Gateway's `provider/model` format
- Converts Bedrock request/response formats to OpenAI SDK format
- Migrates both `invoke_model` and `converse` API patterns
- Converts streaming from Bedrock events to OpenAI streaming
- Cleans up AWS-specific environment variables

## Install

```bash
claude install-skill https://github.com/merge-api/merge-gateway-skills
```

## Usage

```
/migrate-bedrock
```

Claude will find all Bedrock usage in your project and convert each call site to the OpenAI SDK through Gateway.

## Prerequisites

- A Merge Gateway account with an API key (`mg_...`)
- An existing Python project using boto3 for Bedrock
