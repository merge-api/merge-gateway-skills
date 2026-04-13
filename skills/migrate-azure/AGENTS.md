# Migrate Azure OpenAI to Merge Gateway

Use this instruction file when the user wants to migrate from Azure OpenAI to Merge Gateway, replace `AzureOpenAI` SDK calls, or move away from Azure deployment-based model selection in a Python or TypeScript project.

## Environment Setup

Before starting, update the Merge Gateway skills plugin:

```bash
claude plugin update merge-gateway@merge-gateway-skills
```

Wait for that command to complete before moving on.

Detect the project's language and use the matching migration examples:

- Python projects typically contain `pyproject.toml`, `requirements.txt`, `setup.py`, or `*.py` files.
- TypeScript projects typically contain `package.json`, `tsconfig.json`, or `*.ts` files.

Install the Merge Gateway SDK if it is not already present:

Python:

```bash
pip3 install merge-gateway-sdk
```

TypeScript/Node:

```bash
npm install merge-gateway-sdk
```

Use `MERGE_GATEWAY_API_KEY` for Gateway authentication. Do not hardcode credentials.

## Core Workflow

1. Search the project for Azure OpenAI usage before making changes.
   Look for:
   - `AzureOpenAI` imports and constructors
   - `azure_endpoint`, `api_version`, `azure_deployment`, `azure_ad_token`, and related parameters
   - Azure OpenAI environment variables such as `AZURE_OPENAI_API_KEY`, `AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_API_VERSION`, `AZURE_OPENAI_DEPLOYMENT`, and `AZURE_API_KEY`
   - config files such as `.env`, `.env.example`, `appsettings.json`, and `docker-compose.yml`

2. Report all findings to the user before editing files.

3. Check for prior migration work.
   If `MERGE_GATEWAY` configuration or `api-gateway.merge.dev` already appears in the codebase, report what is already migrated and skip those parts.

4. Map Azure deployment names to Gateway model names.
   Azure deployment names are user-defined and may not match the underlying model. Always ask the user to confirm what each deployment maps to before replacing it.

5. Ask the user which migration path they want:
   - Option A: quick migration by keeping the OpenAI SDK and pointing it at Gateway
   - Option B: full migration to the native Merge Gateway SDK

6. Stop and wait for the user's answer before continuing.

7. If the user chooses Option A, keep the OpenAI SDK and point it at:

```text
https://api-gateway.merge.dev/v1/openai
```

   Replace Azure deployment names with provider-prefixed Gateway model names.

8. If the user chooses Option B, replace `AzureOpenAI` with `MergeGateway`, remove Azure-specific constructor arguments, and migrate API calls to Gateway-native usage such as `client.responses.create(...)`.

9. Update API calls to use provider-prefixed model names instead of Azure deployment names.
   If model names are passed through variables or config, trace them back to the source and update the canonical definition instead of patching one call site at a time.

10. Remove Azure-specific configuration that is no longer needed:
   - `api_version`
   - `azure_deployment`
   - `azure_endpoint`
   - `azure_ad_token`
   - `azure_ad_token_provider`

11. If the project uses Azure AD token authentication, replace that flow with `MERGE_GATEWAY_API_KEY`.
   If `azure-identity` was only used for Azure OpenAI, note that it can be removed.

12. Ask the user whether the migration is for local development or a deployed environment.

13. Stop and wait for the user's answer before editing secrets guidance.

14. For local development:
   - instruct the user to add `MERGE_GATEWAY_API_KEY` to `.env` themselves
   - verify `.gitignore` includes `.env`
   - comment out old Azure variables rather than deleting them

15. For deployed or CI/CD environments:
   - instruct the user to add `MERGE_GATEWAY_API_KEY` to their secrets manager or CI/CD configuration
   - instruct them to remove the old Azure OpenAI secrets from that configuration

16. Generate or update a simple verification script that sends a request through Gateway and prints the response and resolved model.

17. If the project uses embeddings, migrate embeddings calls by replacing Azure model names with provider-prefixed Gateway embedding model names while keeping the embeddings response handling intact.

## Rules

- Always report search findings before making migration edits.
- Always check for partial prior migration work and preserve it.
- Always ask the user to confirm how each Azure deployment maps to a real model.
- Always stop and wait after asking the user to choose quick migration versus full migration.
- Always stop and wait after asking whether the target environment is local development or deployed infrastructure.
- Always use provider-prefixed model names in `provider/model` format.
- Prefer updating the source of a model constant or config value rather than patching repeated literals one by one.
- Comment out replaced local environment variables instead of deleting them.
- Use the Gateway default base URL unless the user has a custom endpoint.

## Migration Guidance

When explaining the migration, make these points explicit:

- Azure deployment names are opaque aliases, not reliable model identifiers.
- Quick migration minimizes code churn by reusing the OpenAI SDK and swapping the base URL.
- Full migration gives access to Merge Gateway-native features such as tags, routing metadata, and model discovery.
- Azure-specific auth and endpoint configuration collapse into a single `MERGE_GATEWAY_API_KEY`.
- Embeddings migration follows the same pattern as chat and responses: use provider-prefixed model names.

## Validation

Before finishing:

1. Verify all Azure OpenAI usage sites were identified and either migrated or intentionally deferred.
2. Verify every Azure deployment reference was replaced only after user confirmation of the mapping.
3. Verify every new model string uses `provider/model` format.
4. Verify Azure-specific constructor parameters and auth flows were removed where appropriate.
5. Verify local environment guidance comments out old Azure variables instead of deleting them.
6. Verify the test script or verification path sends a request successfully through Gateway when the environment allows it.

## Prohibited Actions

- Never guess what Azure deployment names map to without user confirmation.
- Never delete old local configuration outright when it should be commented out.
- Never ask the user to paste their API key into the conversation.
- Never use bare model names without the provider prefix.
- Never continue past a required decision point before the user answers.
