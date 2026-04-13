# Integrate Merge Gateway

Use this instruction file when the user wants to add Merge Gateway to an existing project, replace direct provider SDK usage with Gateway, or update configuration so the project routes model calls through Merge Gateway.

## Environment Setup

Before starting, update the Merge Gateway skills plugin:

```bash
claude plugin update merge-gateway@merge-gateway-skills
```

Wait for that command to complete before moving on.

Detect the project's language and use the matching integration path:

- Python projects typically contain `requirements.txt`, `pyproject.toml`, `Pipfile`, `setup.py`, or `*.py` files.
- TypeScript/Node projects typically contain `package.json`, `tsconfig.json`, or `*.ts` files.

Install the Merge Gateway SDK in the detected language:

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

1. Detect the existing stack and LLM integration before making changes.
   Search for:
   - Python dependencies such as `openai`, `anthropic`, `langchain`, `litellm`, `boto3`, and `google-generativeai`
   - TypeScript dependencies such as `openai`, `@anthropic-ai/sdk`, `langchain`, `litellm`, `@google/generative-ai`, and `@azure/openai`
   - client constructors such as `OpenAI(`, `new OpenAI({`, `Anthropic(`, `new Anthropic({`, `AzureOpenAI(`, and `boto3.client('bedrock'`

2. Report what you found to the user before proceeding.

3. Check whether `MERGE_GATEWAY_API_KEY` is already set in the environment or a `.env` file.
   If it is already configured, confirm that with the user and skip duplicate credential setup.

4. If `MERGE_GATEWAY_API_KEY` is not configured, tell the user to create or copy their API key from:

```text
https://gateway.merge.dev
```

5. Ask the user whether they are setting this up for local development or a deployed environment.

6. Stop and wait for the user's answer before continuing with credential setup.

7. For local development:
   - instruct the user to add `MERGE_GATEWAY_API_KEY` to `.env` themselves
   - verify `.gitignore` includes `.env`
   - if `.gitignore` does not include `.env`, add it and warn that `.env` files must not be committed

8. For deployed or CI/CD environments:
   - instruct the user to add `MERGE_GATEWAY_API_KEY` to their secrets manager or platform environment configuration
   - do not create a `.env` file for deployed environments

9. Install `merge-gateway-sdk` in the project if it is not already present.

10. Replace existing LLM client constructors with `MergeGateway` in the detected language.

11. Update API calls to use Gateway's unified responses format with `client.responses.create(...)`.
    Convert chat-style `messages` arrays to `input` items with `type`, `role`, and `content`.

12. Update model names to provider-prefixed `provider/model` format.

13. Search for centralized configuration before making scattered edits.
    Look for:
    - Python config patterns such as `config.py`, `settings.py`, `.env` loading, and `pydantic.BaseSettings`
    - TypeScript config patterns such as `config.ts`, `settings.ts`, DI containers, and centralized `process.env` wrappers

14. If centralized config exists, update Gateway settings there and have call sites reference that config.
    If no centralized config exists, update each call site directly.

15. Verify the environment setup.
    The SDK defaults to:

```text
https://api-gateway.merge.dev/v1
```

    Do not add `base_url` or `baseUrl` unless the user has a custom gateway endpoint.

16. If old provider API key variables exist locally, comment them out instead of deleting them.

17. Generate a test script in the project that sends a request through Gateway and prints both the response text and the resolved model.

18. Ask the user whether they want to run the test script.

19. Stop and wait for the user's answer before running the verification script.

20. If the user approves, run the test script and confirm it prints a successful response.
    If it fails, check whether `MERGE_GATEWAY_API_KEY` is set correctly and whether `.env` loading support such as `dotenv` or `python-dotenv` is installed when needed.

21. If the project uses embeddings, migrate embeddings calls by prefixing the model name with `openai/` and keep the existing embeddings response handling.

22. If the project uses streaming, preserve streaming behavior and handle accumulated text chunks correctly by printing only the newly added portion of each streamed text update.

23. Show the user how to handle Gateway exceptions with SDK-native error classes such as `AuthenticationError`, `BadRequestError`, `RateLimitError`, and `APIError`.

## Model Guidance

Use provider-prefixed model names when moving traffic through Gateway. Apply these common mappings:

- `gpt-4o` -> `openai/gpt-4o`
- `gpt-4o-mini` -> `openai/gpt-4o-mini`
- `gpt-4-turbo` -> `openai/gpt-4-turbo`
- `gpt-3.5-turbo` -> `openai/gpt-3.5-turbo`
- `claude-sonnet-4-6-20250514` -> `anthropic/claude-sonnet-4-6-20250514`
- `claude-3-5-haiku-20241022` -> `anthropic/claude-3-5-haiku-20241022`
- `gemini-2.0-flash` -> `google/gemini-2.0-flash`
- `text-embedding-3-small` -> `openai/text-embedding-3-small`

## Error Handling

Use Merge Gateway SDK exceptions when wiring verification or production code:

- `BadRequestError` for invalid request shape or parameters
- `AuthenticationError` for missing or invalid API keys
- `APIError` with status `402` for exhausted free-tier budget
- `NotFoundError` for missing models or endpoints
- `RateLimitError` for throttling

Treat all exceptions as `MergeGatewayError`-compatible objects with message and status metadata when surfacing failures to the user.

## Rules

- Always report stack detection findings before making integration edits.
- Always check for existing `MERGE_GATEWAY_API_KEY` or `MergeGateway` usage and preserve already-migrated files.
- Always stop and wait after asking whether the target environment is local development or deployed infrastructure.
- Always stop and wait after asking whether to run the verification script.
- Always use provider-prefixed model names in `provider/model` format.
- Prefer updating centralized configuration over patching repeated literals one call site at a time.
- Comment out replaced local provider secrets instead of deleting them.
- Use the Gateway default base URL unless the user explicitly has a custom endpoint.
- Keep embeddings response handling unchanged when only the model name needs migration.
- Preserve streaming behavior by handling accumulated stream chunks correctly.

## Validation

Before finishing:

1. Verify the project's stack and current LLM integration were identified and reported.
2. Verify `MERGE_GATEWAY_API_KEY` is configured in the correct place for the target environment.
3. Verify the Merge Gateway SDK is installed or clearly note if installation could not be completed.
4. Verify all updated model strings use `provider/model` format.
5. Verify old local provider keys were commented out instead of deleted.
6. Verify the generated test script exists and matches the detected language.
7. If the user approved running verification, verify the script produced a visible successful response or report the exact failure condition.
8. Verify embeddings and streaming integrations were preserved correctly where applicable.

## Prohibited Actions

- Never ask the user to paste their API key into the conversation.
- Never continue past a required decision point before the user answers.
- Never create a `.env` file for deployed or CI/CD environments.
- Never delete old local configuration outright when it should be commented out.
- Never use bare model names without the provider prefix.
- Never add `base_url` or `baseUrl` unless the user has a custom gateway endpoint.
- Never claim verification passed unless the test script actually produced a visible successful response.
