# Migrate from AWS Bedrock

Replace boto3 Bedrock calls with the Merge Gateway SDK.

## What it does

- Finds all AWS Bedrock usage (`invoke_model`, `converse`, streaming calls)
- Maps Bedrock model IDs to Gateway's `provider/model` format
- Converts Bedrock request/response formats to Merge Gateway SDK format
- Migrates both `invoke_model` and `converse` API patterns
- Converts streaming from Bedrock events to Merge Gateway SDK streaming
- Cleans up AWS-specific environment variables

## Install

```bash
claude plugin marketplace add merge-api/merge-gateway-skills
claude plugin install merge-gateway
```

## Usage

```
/migrate-bedrock
```

Claude will find all Bedrock usage in your project and convert each call site to the Merge Gateway SDK.

## Prerequisites

- A Merge Gateway account with an API key (`mg_...`)
- An existing Python project using boto3 for Bedrock
