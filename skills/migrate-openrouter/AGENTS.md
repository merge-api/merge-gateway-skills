# Migrate from OpenRouter to Merge Gateway

Use this instruction file when the user wants to switch from OpenRouter to Merge Gateway, replace OpenRouter configuration, or move OpenRouter-based SDK usage to Gateway in a Python or TypeScript project.

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

1. Search the project for OpenRouter usage before making changes.
   Look for:
   - `openrouter.ai` URLs, especially `openrouter.ai/api/v1`
   - `OPENROUTER_API_KEY`, `OPENROUTER_BASE_URL`, and `OPENROUTER_API_BASE`
   - `HTTP-Referer` and `X-Title` headers
   - config files such as `.env`, `.env.example`, `.env.local`, `docker-compose.yml`, and `config.yaml`
   - `openai` SDK usage that is currently pointed at OpenRouter

2. Report all findings to the user before editing files.

3. Check for prior migration work.
   If `MERGE_GATEWAY` configuration or `api-gateway.merge.dev` already appears in the codebase, report what is already migrated and skip those parts.

4. Ask the user which migration path they want:
   - Option A: quick migration by keeping the OpenAI SDK and pointing it at Gateway
   - Option B: full migration to the native Merge Gateway SDK

5. Stop and wait for the user's answer before continuing.

6. If the user chooses Option A, keep the OpenAI SDK and point it at:

```text
https://api-gateway.merge.dev/v1/openai
```

   Replace `OPENROUTER_API_KEY` with `MERGE_GATEWAY_API_KEY`.

7. If the user chooses Option B, replace the OpenAI client configured for OpenRouter with `MergeGateway` and use `MERGE_GATEWAY_API_KEY`.

8. Review all model names after the client migration.
   OpenRouter already uses `provider/model` format, so many model names can stay unchanged, but confirm mappings with the user for less common or provider-specific models before replacing them.

9. Remove OpenRouter-specific request parameters from API calls:
   - `transforms`
   - `route`
   - `provider`
   - `models`

10. When migrating OpenAI-style chat calls to Gateway-native responses usage, convert message arrays to `input` items for `client.responses.create(...)`.

11. Remove OpenRouter-specific headers:
    - `HTTP-Referer`
    - `X-Title`

12. If `default_headers` or equivalent configuration contains non-OpenRouter headers, preserve those headers.

13. Ask the user whether the migration target is local development or a deployed environment.

14. Stop and wait for the user's answer before changing environment guidance.

15. For local development:
    - instruct the user to add `MERGE_GATEWAY_API_KEY` to `.env` themselves
    - verify `.gitignore` includes `.env`
    - comment out old OpenRouter environment variables rather than deleting them

16. For deployed or CI/CD environments:
    - instruct the user to add `MERGE_GATEWAY_API_KEY` to their secrets manager or CI/CD configuration
    - instruct them to remove the old OpenRouter secrets from that configuration

17. Generate or update a simple verification script that sends a request through Gateway and prints both the response text and the resolved model.

## Rules

- Always report OpenRouter search findings before making migration edits.
- Always check for partial prior migration work and preserve it.
- Always stop and wait after asking the user to choose quick migration versus full migration.
- Always stop and wait after asking whether the target environment is local development or deployed infrastructure.
- Always use provider-prefixed model names in `provider/model` format.
- Prefer updating shared model constants or config values instead of patching repeated literals one by one.
- Comment out replaced local OpenRouter environment variables instead of deleting them.
- Use the Gateway default base URL unless the user has a custom endpoint.
- Remove only OpenRouter-specific headers and parameters; preserve unrelated request configuration.
- Explain that Gateway routing and fallback behavior should be configured through Gateway policies rather than OpenRouter request parameters.

## Model Guidance

Use these mappings as defaults when the project uses common OpenRouter model names:

- `openai/gpt-4o` -> `openai/gpt-4o`
- `anthropic/claude-3.5-sonnet` -> `anthropic/claude-sonnet-4-6-20250514`
- `google/gemini-pro` -> `google/gemini-2.0-flash`
- `meta-llama/llama-3.1-70b-instruct` -> `meta-llama/llama-3.1-70b-instruct`

Confirm uncommon model mappings with the user before changing them, because some OpenRouter models may not be available through Gateway.

## Feature Mapping

Map OpenRouter features to Gateway behavior like this:

- `route: "fallback"` -> Gateway routing policies with fallback strategies
- `transforms` -> remove; do not carry this forward
- `provider.order` -> Gateway routing policy provider priorities
- OpenRouter usage dashboard -> Gateway usage dashboard
- OpenRouter per-model billing -> unified billing through Gateway

## Validation

Before finishing:

1. Verify every OpenRouter usage site was identified and either migrated or intentionally deferred.
2. Verify all retained or updated model strings use `provider/model` format.
3. Verify OpenRouter-specific headers and request parameters were removed without deleting unrelated configuration.
4. Verify old OpenRouter environment variables are commented out locally instead of deleted.
5. Verify the test script or verification path sends a request successfully through Gateway when the environment allows it.

## Prohibited Actions

- Never make migration edits before reporting search findings to the user.
- Never continue past a required decision point before the user answers.
- Never ask the user to paste their API key into the conversation.
- Never delete old local configuration outright when it should be commented out.
- Never remove non-OpenRouter headers accidentally when cleaning up `default_headers`.
- Never keep OpenRouter-only parameters such as `transforms`, `route`, `provider`, or `models` and claim they still work through Gateway unchanged.
