# Migrate LiteLLM to Merge Gateway

Use this instruction file when the user wants to replace LiteLLM proxy usage, migrate off direct `litellm` Python library calls, or move a self-hosted LiteLLM setup to Merge Gateway.

## Environment Setup

Before starting, update the Merge Gateway skills plugin:

```bash
claude plugin update merge-gateway@merge-gateway-skills
```

Wait for that command to complete before moving on.

Detect the project's language and LiteLLM integration style before editing:

- Python projects may use direct `litellm` imports and function calls.
- Python or TypeScript projects may use the OpenAI SDK pointed at a LiteLLM proxy.
- LiteLLM proxy infrastructure may also appear in Docker, Kubernetes, or YAML config files.

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

1. Search the project for LiteLLM usage before making changes.
   Look for:
   - proxy URLs such as `localhost:4000`, `localhost:8000`, or internal `/chat/completions` endpoints
   - `LITELLM_API_KEY`, `LITELLM_BASE_URL`, and `LITELLM_PROXY_URL`
   - `litellm_config.yaml`, `proxy_config.yaml`, and `config.yaml` with `model_list` or `litellm_settings`
   - direct imports such as `import litellm`, `from litellm import completion`, and `from litellm import acompletion`
   - direct calls such as `litellm.completion(`, `litellm.acompletion(`, and `litellm.embedding(`
   - `litellm` dependencies in Python dependency files
   - Docker or Kubernetes LiteLLM infrastructure definitions

2. Report all findings to the user before editing files.

3. Check for prior migration work.
   If `MERGE_GATEWAY` configuration or `api-gateway.merge.dev` already appears in the codebase, report what is already migrated and skip those parts.

4. Determine which migration path applies:
   - Path A: LiteLLM proxy client through the OpenAI SDK
   - Path B: direct LiteLLM Python library usage

5. If the search results do not make the path clear, ask the user whether they are using the LiteLLM proxy or the LiteLLM Python library.

6. Stop and wait for the user's answer if the path is ambiguous.

7. For Path A, ask the user which migration path they want:
   - Option A: quick migration by keeping the OpenAI SDK and pointing it at Gateway
   - Option B: full migration to the native Merge Gateway SDK

8. Stop and wait for the user's answer before continuing with Path A migration.

9. If the user chooses Path A Option A, keep the OpenAI SDK and point it at:

```text
https://api-gateway.merge.dev/v1/openai
```

   Replace LiteLLM proxy credentials with `MERGE_GATEWAY_API_KEY`.

10. If the user chooses Path A Option B, replace the OpenAI client configured for LiteLLM with `MergeGateway` and migrate chat calls to `client.responses.create(...)`.

11. For Path B, replace `litellm.completion()` and related calls with `MergeGateway` client calls.

12. For async LiteLLM migrations, replace `litellm.acompletion(...)` with synchronous `MergeGateway` client calls wrapped in an executor such as `asyncio.to_thread(...)` until a native async client is available.

13. For streaming LiteLLM migrations, preserve streaming behavior by printing only the newly appended portion of each accumulated text chunk from Gateway streaming responses.

14. For embeddings migrations, replace `litellm.embedding(...)` with `client.embeddings.create(...)` and keep the existing embeddings response handling.

15. Update model names everywhere they are defined.
    Handle these cases:
    - already-prefixed models that can transfer directly
    - bare model names that need a provider prefix added
    - LiteLLM-specific formats such as `bedrock/...` that require remapping

16. If the project uses Azure deployment-style LiteLLM model references such as `azure/my-deployment`, ask the user what real model each deployment maps to before changing it.

17. Remove LiteLLM-specific configuration and parameters such as:
    - `litellm.set_verbose`
    - `litellm.success_callback`
    - `litellm.failure_callback`
    - `litellm.api_base`
    - `litellm.drop_params`
    - `litellm.modify_params`

18. Ask the user whether the migration target is local development or a deployed environment.

19. Stop and wait for the user's answer before changing environment guidance.

20. For local development:
    - instruct the user to add `MERGE_GATEWAY_API_KEY` to `.env` themselves
    - verify `.gitignore` includes `.env`
    - comment out old LiteLLM and provider environment variables rather than deleting them

21. For deployed or CI/CD environments:
    - instruct the user to add `MERGE_GATEWAY_API_KEY` to their secrets manager or CI/CD configuration
    - instruct them to remove old LiteLLM and provider secrets from that configuration

22. Ask the user before removing LiteLLM infrastructure or dependency files.
    Flag these for cleanup instead of deleting them automatically:
    - `litellm` dependency entries
    - LiteLLM proxy Docker services
    - LiteLLM Kubernetes deployments and services
    - `litellm_config.yaml` and `proxy_config.yaml`

23. Generate or update a simple verification script that sends a request through Gateway and prints both the response text and the resolved model.

## Model Guidance

Apply these model migration rules:

- `openai/gpt-4o` -> `openai/gpt-4o`
- `anthropic/claude-3-5-sonnet-20241022` -> `anthropic/claude-3-5-sonnet-20241022`
- `gpt-4o` -> `openai/gpt-4o`
- `gpt-4o-mini` -> `openai/gpt-4o-mini`
- `claude-3-5-haiku-20241022` -> `anthropic/claude-3-5-haiku-20241022`
- `claude-sonnet-4-6-20250514` -> `anthropic/claude-sonnet-4-6-20250514`
- `bedrock/anthropic.claude-3-5-sonnet-20241022-v2:0` -> `anthropic/claude-3-5-sonnet-20241022`

Always use `provider/model` format after migration.
Ask the user before changing Azure deployment aliases such as `azure/my-deployment`, because those names do not identify the underlying model.

## Feature Mapping

Map LiteLLM features to Gateway behavior like this:

- LiteLLM config-based model routing -> Gateway routing policies
- LiteLLM `fallbacks` -> Gateway fallback strategies
- LiteLLM rate limiting config -> Gateway rate limiting
- LiteLLM spend tracking and budgets -> Gateway usage dashboard and budget controls
- LiteLLM logging and callbacks -> Gateway logs and analytics
- LiteLLM load balancing through `model_list` -> Gateway routing policies with load balancing
- LiteLLM key generation flows -> Gateway API key management

## Rules

- Always report LiteLLM search findings before making migration edits.
- Always check for partial prior migration work and preserve it.
- Always stop and wait when the migration path is ambiguous.
- Always stop and wait after asking the user to choose quick migration versus full migration for proxy-based integrations.
- Always stop and wait after asking whether the target environment is local development or deployed infrastructure.
- Always ask the user before removing LiteLLM infrastructure or dependency files.
- Always use provider-prefixed model names in `provider/model` format.
- Comment out replaced local LiteLLM and provider environment variables instead of deleting them.
- Use the Gateway default base URL unless the user has a custom endpoint.
- Remove the `litellm` dependency only after all LiteLLM imports are gone.

## Validation

Before finishing:

1. Verify every LiteLLM usage site was identified and either migrated or intentionally deferred.
2. Verify the correct migration path was applied for proxy-based or direct-library usage.
3. Verify all updated model strings use `provider/model` format.
4. Verify async, streaming, and embeddings behavior were preserved correctly where applicable.
5. Verify LiteLLM-specific parameters and callbacks were removed or replaced appropriately.
6. Verify old local LiteLLM and provider keys were commented out instead of deleted.
7. Verify no infrastructure or dependency cleanup was performed without user approval.
8. Verify the test script or verification path sends a request successfully through Gateway when the environment allows it.

## Prohibited Actions

- Never make migration edits before reporting search findings to the user.
- Never continue past a required decision point before the user answers.
- Never ask the user to paste their API key into the conversation.
- Never delete old local configuration outright when it should be commented out.
- Never delete LiteLLM proxy infrastructure automatically.
- Never remove the `litellm` dependency before all LiteLLM imports are removed.
- Never use bare model names without the provider prefix.
