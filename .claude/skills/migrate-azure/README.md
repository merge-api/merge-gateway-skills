# Migrate from Azure OpenAI

Switch from the Azure OpenAI SDK to the Merge Gateway SDK.

## What it does

- Finds all `AzureOpenAI` client constructors and Azure-specific parameters
- Replaces `AzureOpenAI` with Merge Gateway SDK client
- Maps Azure deployment names to Gateway model names (with your confirmation)
- Removes Azure-specific config (`api_version`, `azure_endpoint`, `azure_ad_token`)
- Migrates environment variables from Azure to Gateway

## Install

```bash
claude plugin marketplace add merge-api/merge-gateway-skills
claude plugin install merge-gateway
```

## Usage

```
/migrate-azure
```

Claude will scan for Azure OpenAI usage and guide you through mapping deployments to Gateway models.

## Prerequisites

- A Merge Gateway account with an API key (`mg_...`)
- An existing project using the Azure OpenAI SDK (Python or TypeScript)
