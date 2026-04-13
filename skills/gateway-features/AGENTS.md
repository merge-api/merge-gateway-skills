# Gateway Features

Use this instruction file after a user already has a working Merge Gateway integration and wants to explore advanced Gateway features such as tags, project tracking, routing metadata, multi-model routing, extended thinking, token usage tracking, or embeddings.

## Environment Setup

Before starting, update the Merge Gateway skills plugin:

```bash
claude plugin update merge-gateway@merge-gateway-skills
```

Wait for that command to complete before moving on.

Use the language already present in the project:

- Python projects typically contain `pyproject.toml`, `requirements.txt`, `setup.py`, or `*.py` files.
- TypeScript projects typically contain `package.json`, `tsconfig.json`, or `*.ts` files.

Install the Merge Gateway SDK if it is missing:

Python:

```bash
pip3 install merge-gateway-sdk
```

TypeScript/Node:

```bash
npm install merge-gateway-sdk
```

Use `MERGE_GATEWAY_API_KEY` from environment configuration. Do not hardcode credentials.

## Core Workflow

1. Verify the user already has a working Gateway integration.
   Check for:
   - `MERGE_GATEWAY_API_KEY` in `.env` or equivalent environment configuration
   - `merge-gateway-sdk` in dependencies
   - existing `MergeGateway` client usage in code

2. If the integration is not in place, stop and direct the user to complete the base Gateway implementation first.

3. Detect whether the project is Python or TypeScript and show examples in the matching language.
   If both are present, ask the user which language they want to use before making changes.

4. Introduce tags when the user wants request organization, cost attribution, or dashboard filtering.
   Show how to pass a `tags` array to `client.responses.create(...)` and suggest practical keys such as:
   - `feature`
   - `team`
   - `environment`
   - `user_tier`
   - `experiment`

5. Introduce `project_id` when the user needs per-project grouping, usage tracking, or budgeting.

6. Introduce routing metadata when the user wants to debug routing behavior, analyze costs, or understand which model actually served a request.
   Show how to enable `include_routing_metadata` and inspect fields such as:
   - `policy_name`
   - `strategy`
   - `model_requested`
   - `model_used`
   - `cost_usd`
   - `intelligent_routing_used`
   - `complexity_score`
   - latency breakdowns

7. Show multi-model routing by reusing the same client with different provider-prefixed model strings such as:
   - `openai/gpt-4o`
   - `anthropic/claude-sonnet-4-6-20250514`
   - `google/gemini-2.0-flash`

8. Show model discovery when the user needs to browse supported models, filter by provider, filter by capability, or inspect model pricing.

9. Introduce extended thinking only for supported Anthropic Claude models and only when the user is working on reasoning-heavy tasks.
   Show how to pass a `thinking` object and how to inspect `thinking` blocks in the response.

10. Show token usage tracking by reading `response.usage` from standard responses.

11. Show embeddings through `client.embeddings.create(...)` when the user needs semantic search, retrieval, clustering, or similarity workflows.

12. After covering the SDK features, direct the user to the Gateway dashboard for:
   - tags
   - routing policies
   - budgets
   - usage analytics
   - API key management

## Rules

- Always verify the user has completed the base Gateway integration before teaching advanced features.
- Always use the project's existing language unless the user explicitly asks to switch.
- Always use provider-prefixed model names in `provider/model` format.
- Only set `base_url` or `baseUrl` when the user has a custom gateway endpoint.
- Suggest tags where they add operational value instead of adding them blindly to every call.
- Use routing metadata for debugging, investigation, or development workflows.
- Prefer minimal, relevant feature examples over dumping every feature at once.
- Show concrete code in the user's language for any feature you recommend.

## Feature Guidance

When explaining advanced features, make the operational value explicit:

- Tags help organize requests and break down cost or usage by feature, team, environment, or experiment.
- `project_id` groups usage for a product, workspace, or deployment surface.
- Routing metadata explains what policy ran, what model actually served the request, what it cost, and where time was spent.
- Multi-model routing lets the same client talk to multiple providers by changing the model string.
- Extended thinking is useful for complex reasoning but should only be used on supported Anthropic models.
- Token usage tracking helps users monitor prompt and completion cost.
- Embeddings support retrieval and vector-based workflows through the same Gateway account.

## Validation

Before finishing:

1. Verify the examples match the user's language and current integration state.
2. Verify every model string uses `provider/model` format.
3. Verify advanced features are only recommended when they fit the user's use case.
4. Verify routing metadata is not left enabled by default in production-oriented examples unless the user specifically wants it.
5. Verify any dashboard guidance matches the code-level features you introduced.

## Prohibited Actions

- Never teach advanced features before confirming the base Gateway setup exists.
- Never hardcode API keys in sample code.
- Never use bare model names without the provider prefix.
- Never recommend `include_routing_metadata` as a default production setting.
- Never add tags to every request without explaining why they matter.
