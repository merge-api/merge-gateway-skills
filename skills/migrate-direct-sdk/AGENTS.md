# Migrate Direct Provider SDKs to Merge Gateway

Use this instruction file when the user wants to replace direct OpenAI, Anthropic, or Google SDK calls with Merge Gateway in a Python or TypeScript project.

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

1. Search the project for direct provider SDK usage before making changes.
   Look for:
   - OpenAI imports, constructors, and `OPENAI_API_KEY`
   - Anthropic imports, constructors, and `ANTHROPIC_API_KEY`
   - Google Generative AI imports, constructors, `GOOGLE_API_KEY`, and `GEMINI_API_KEY`

2. Report all findings to the user before editing files.

3. Check for prior migration work.
   If `MERGE_GATEWAY` configuration or `api-gateway.merge.dev` already appears in the codebase, report what is already migrated and skip those parts.

4. If OpenAI usage is detected, ask the user which migration path they want:
   - Option A: quick migration by keeping the OpenAI SDK and pointing it at Gateway
   - Option B: full migration to the native Merge Gateway SDK

5. Stop and wait for the user's answer before continuing with any OpenAI migration.

6. If the user chooses OpenAI Option A, keep the OpenAI SDK and point it at:

```text
https://api-gateway.merge.dev/v1/openai
```

   Replace `OPENAI_API_KEY` with `MERGE_GATEWAY_API_KEY` and prefix every model name with `openai/`.

7. If the user chooses OpenAI Option B, replace `OpenAI` with `MergeGateway`, use `MERGE_GATEWAY_API_KEY`, and prefix every model name with `openai/`.

8. If Anthropic usage is detected, replace the Anthropic SDK with `MergeGateway` by default.
   As an alternative only if needed, the Anthropic SDK can be kept and pointed at:

```text
https://api-gateway.merge.dev
```

   Do not append `/v1` when keeping the Anthropic SDK. Prefix every Anthropic model name with `anthropic/`.

9. If Google Generative AI usage is detected, replace it with the Merge Gateway SDK.
   Migrate calls such as `generate_content(...)`, `chat.send_message(...)`, and `start_chat()` flows to `client.responses.create(...)` and maintain conversation state in the `input` array.

10. Map Google model names to provider-prefixed Gateway model names:
    - `gemini-pro` -> `google/gemini-2.0-flash`
    - `gemini-pro-vision` -> `google/gemini-2.0-flash`
    - `gemini-1.5-pro` -> `google/gemini-1.5-pro`
    - `gemini-1.5-flash` -> `google/gemini-1.5-flash`

11. If the project uses OpenAI embeddings, migrate embeddings calls by prefixing the model name with `openai/` and keep the existing response handling.

12. Ask the user whether the migration target is local development or a deployed environment.

13. Stop and wait for the user's answer before changing environment guidance.

14. For local development:
    - instruct the user to add `MERGE_GATEWAY_API_KEY` to `.env` themselves
    - verify `.gitignore` includes `.env`
    - comment out old provider variables rather than deleting them

15. For deployed or CI/CD environments:
    - instruct the user to add `MERGE_GATEWAY_API_KEY` to their secrets manager or CI/CD configuration
    - instruct them to remove the old provider secrets from that configuration

16. Generate or update a simple verification script that sends a request through Gateway and prints both the response text and the resolved model.

## Rules

- Always report provider search findings before making migration edits.
- Always check for partial prior migration work and preserve it.
- Always stop and wait after asking the user to choose quick migration versus full migration for OpenAI.
- Always stop and wait after asking whether the target environment is local development or deployed infrastructure.
- Always use provider-prefixed model names in `provider/model` format.
- Prefer updating shared model constants or config values instead of patching repeated literals one by one.
- Comment out replaced local provider environment variables instead of deleting them.
- Use the Gateway default base URL unless the user has a custom endpoint.
- When keeping the Anthropic SDK as a compatibility path, use `https://api-gateway.merge.dev` without `/v1`.
- Explain that Google SDK migration changes API shape and requires moving to the Gateway SDK interface.

## Model Mapping

Use these OpenAI model mappings when replacing bare OpenAI names:

- `gpt-4o` -> `openai/gpt-4o`
- `gpt-4o-mini` -> `openai/gpt-4o-mini`
- `gpt-4-turbo` -> `openai/gpt-4-turbo`
- `gpt-3.5-turbo` -> `openai/gpt-3.5-turbo`
- `o1` -> `openai/o1`
- `o1-mini` -> `openai/o1-mini`
- `o3-mini` -> `openai/o3-mini`

Use these Anthropic model mappings when replacing bare Anthropic names:

- `claude-sonnet-4-6-20250514` -> `anthropic/claude-sonnet-4-6-20250514`
- `claude-3-5-haiku-20241022` -> `anthropic/claude-3-5-haiku-20241022`
- `claude-3-opus-20240229` -> `anthropic/claude-3-opus-20240229`

## Validation

Before finishing:

1. Verify every direct provider SDK usage site was identified and either migrated or intentionally deferred.
2. Verify all new model strings use `provider/model` format.
3. Verify old provider environment variables are commented out locally instead of deleted.
4. Verify any retained compatibility SDK uses the correct Gateway base URL.
5. Verify the test script or verification path sends a request successfully through Gateway when the environment allows it.

## Prohibited Actions

- Never make migration edits before reporting search findings to the user.
- Never continue past a required decision point before the user answers.
- Never ask the user to paste their API key into the conversation.
- Never delete old local configuration outright when it should be commented out.
- Never use bare model names without the provider prefix.
- Never keep Google's SDK API shape and claim it works unchanged through Gateway.
