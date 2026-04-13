# Migrate AWS Bedrock to Merge Gateway

Use this instruction file when the user wants to replace AWS Bedrock `boto3` calls with Merge Gateway, move off Bedrock-specific request formats, or switch a Python or TypeScript project to Gateway-based model access.

## Environment Setup

Before starting, update the Merge Gateway skills plugin:

```bash
claude plugin update merge-gateway@merge-gateway-skills
```

Wait for that command to complete before moving on.

Detect the project's language and use the matching migration examples:

- Bedrock integrations are commonly in Python through `boto3`.
- The replacement code can be written in Python or TypeScript with `merge-gateway-sdk`.
- If the project is TypeScript-based but Bedrock is accessed through a Python service layer, migrate the actual integration point rather than assuming the outer app language is the only target.

Install the Merge Gateway SDK if it is not already present:

Python:

```bash
pip3 install merge-gateway-sdk
```

TypeScript/Node:

```bash
npm install merge-gateway-sdk
```

Use `MERGE_GATEWAY_API_KEY` for authentication. Do not hardcode credentials.

## Core Workflow

1. Search the project for Bedrock usage before making changes.
   Look for:
   - `boto3.client('bedrock-runtime')`, `boto3.client("bedrock-runtime")`, and `boto3.client('bedrock')`
   - `invoke_model`, `invoke_model_with_response_stream`, `converse`, and `converse_stream`
   - Bedrock model IDs such as `anthropic.claude-3-5-sonnet-20241022-v2:0`, `amazon.titan-text-express-v1`, and `meta.llama3-1-70b-instruct-v1:0`
   - `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_DEFAULT_REGION` when they appear Bedrock-specific
   - `import boto3` and `from botocore`

2. Report all findings to the user before editing files.

3. Check for prior migration work.
   If `MERGE_GATEWAY` configuration or `api-gateway.merge.dev` already appears in the codebase, report what is already migrated and skip those parts.

4. Install `merge-gateway-sdk` if it is not already present in the project's dependencies.

5. Map Bedrock model IDs to Gateway model names.
   Default to 1:1 mappings that preserve behavior rather than silently upgrading models.

6. Ask the user to confirm Bedrock model mappings, especially for uncommon models or models that may not have a Gateway equivalent.

7. Ask the user which migration path they want:
   - Option A: quick migration by using the OpenAI SDK pointed at Gateway
   - Option B: full migration to the native Merge Gateway SDK

8. Stop and wait for the user's answer before continuing.

9. If the user chooses Option A, use the OpenAI SDK pointed at:

```text
https://api-gateway.merge.dev/v1/openai
```

   Keep the migration focused on minimal code churn while switching credentials to `MERGE_GATEWAY_API_KEY`.

10. If the user chooses Option B, replace Bedrock `boto3` calls with `MergeGateway` and migrate to `client.responses.create(...)`.

11. For `invoke_model` migrations, replace provider-specific raw JSON request bodies with Gateway `input` messages and provider-prefixed `model` names.

12. For `converse` and `converse_stream` migrations, convert Bedrock message objects such as `{"content": [{"text": "..."}]}` to Gateway `input` items and map `inferenceConfig` fields to Gateway request parameters.

13. For streaming migrations, preserve streaming behavior by handling accumulated text chunks correctly and printing only the newly appended portion of each streamed response chunk.

14. Ask the user whether `boto3` is used for anything besides Bedrock.

15. Stop and wait for the user's answer before removing `boto3` or AWS credential usage.

16. If `boto3` is only used for Bedrock, remove Bedrock-only dependencies and configuration.
    If `boto3` is used for other AWS services, remove only Bedrock-specific code.

17. Ask the user whether the migration target is local development or a deployed environment.

18. Stop and wait for the user's answer before changing environment guidance.

19. For local development:
    - instruct the user to add `MERGE_GATEWAY_API_KEY` to `.env` themselves
    - verify `.gitignore` includes `.env`
    - comment out old AWS keys instead of deleting them, because they may still be needed for other AWS services

20. For deployed or CI/CD environments:
    - instruct the user to add `MERGE_GATEWAY_API_KEY` to their secrets manager or CI/CD configuration
    - only remove `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` from secrets configuration if they were used solely for Bedrock

21. Generate or update a simple verification script that sends a request through Gateway and prints both the response text and the resolved model.

## Model Mapping

Use these default Bedrock-to-Gateway mappings unless the user confirms a different target:

- `anthropic.claude-3-5-sonnet-20241022-v2:0` -> `anthropic/claude-3-5-sonnet-20241022`
- `anthropic.claude-3-sonnet-20240229-v1:0` -> `anthropic/claude-3-sonnet-20240229`
- `anthropic.claude-3-haiku-20240307-v1:0` -> `anthropic/claude-3-haiku-20240307`
- `anthropic.claude-3-opus-20240229-v1:0` -> `anthropic/claude-3-opus-20240229`
- `meta.llama3-1-70b-instruct-v1:0` -> `meta-llama/llama-3.1-70b-instruct`
- `meta.llama3-1-8b-instruct-v1:0` -> `meta-llama/llama-3.1-8b-instruct`
- `cohere.command-r-plus-v1:0` -> `cohere/command-r-plus`

Treat optional "latest model" upgrades as a separate follow-up, not part of the default migration.
Ask the user before changing `amazon.titan-text-express-v1` or any other model without a clear Gateway equivalent.

## Message Mapping

Apply these Bedrock request mappings when converting calls:

- Bedrock `{"content": [{"text": "..."}]}` -> Gateway `{"type": "message", "content": "..."}`
- Bedrock `system` -> Gateway `input` item with `role: "system"`
- Bedrock `inferenceConfig.maxTokens` -> Gateway `max_output_tokens` or equivalent request parameter
- Bedrock `inferenceConfig.temperature` -> Gateway `temperature`
- Bedrock `inferenceConfig.topP` -> Gateway `top_p`

## Rules

- Always report Bedrock search findings before making migration edits.
- Always check for partial prior migration work and preserve it.
- Always default to 1:1 model mappings unless the user explicitly approves an upgrade.
- Always stop and wait after asking the user to choose quick migration versus full migration.
- Always stop and wait after asking whether `boto3` is used for anything besides Bedrock.
- Always stop and wait after asking whether the target environment is local development or deployed infrastructure.
- Always use provider-prefixed model names in `provider/model` format.
- Comment out replaced local AWS credentials instead of deleting them.
- Use the Gateway default base URL unless the user has a custom endpoint.
- Preserve streaming behavior when migrating `invoke_model_with_response_stream` or `converse_stream`.

## Validation

Before finishing:

1. Verify every Bedrock usage site was identified and either migrated or intentionally deferred.
2. Verify every model mapping was preserved as a 1:1 migration unless the user approved an upgrade.
3. Verify all new model strings use `provider/model` format.
4. Verify Bedrock request bodies and message structures were translated to Gateway request format correctly.
5. Verify `boto3` removal only happened if the user confirmed it was not needed for other AWS services.
6. Verify old local AWS keys were commented out instead of deleted.
7. Verify the test script or verification path sends a request successfully through Gateway when the environment allows it.

## Prohibited Actions

- Never make migration edits before reporting search findings to the user.
- Never continue past a required decision point before the user answers.
- Never silently upgrade models during migration.
- Never ask the user to paste their API key into the conversation.
- Never delete old local AWS configuration outright when it should be commented out.
- Never remove `boto3` if it may still be needed for other AWS services.
- Never use bare model names without the provider prefix.
