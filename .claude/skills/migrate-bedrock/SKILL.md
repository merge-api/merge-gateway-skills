# Migrate AWS Bedrock to Merge Gateway

Migrate from boto3 Bedrock calls to the OpenAI SDK through Merge Gateway. This is a significant API change — the request/response format changes entirely.

## Steps

### 1. Search for Bedrock Usage

Search the project for all AWS Bedrock references:

- **Clients:** `boto3.client('bedrock-runtime')`, `boto3.client("bedrock-runtime")`, `boto3.client('bedrock')`
- **Methods:** `invoke_model`, `invoke_model_with_response_stream`, `converse`, `converse_stream`
- **Model IDs:** Bedrock-format IDs like `anthropic.claude-3-5-sonnet-20241022-v2:0`, `amazon.titan-text-express-v1`, `meta.llama3-1-70b-instruct-v1:0`
- **Env vars:** `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION` (if only used for Bedrock)
- **Imports:** `import boto3`, `from botocore`

Report all findings to the user before making changes.

### 2. Check for Prior Migration

Check if `MERGE_GATEWAY` or `gateway.merge.dev` already exists in the project. If so, report which parts are already migrated and skip those.

### 3. Map Bedrock Model IDs to Gateway Format

Bedrock uses its own model ID format. Map them to Gateway's `provider/model` format:

| Bedrock Model ID | Gateway Model Name |
|---|---|
| `anthropic.claude-3-5-sonnet-20241022-v2:0` | `anthropic/claude-sonnet-4-20250514` |
| `anthropic.claude-3-haiku-20240307-v1:0` | `anthropic/claude-3-5-haiku-20241022` |
| `anthropic.claude-3-opus-20240229-v1:0` | `anthropic/claude-3-opus-20240229` |
| `meta.llama3-1-70b-instruct-v1:0` | `meta-llama/llama-3.1-70b-instruct` |
| `meta.llama3-1-8b-instruct-v1:0` | `meta-llama/llama-3.1-8b-instruct` |
| `amazon.titan-text-express-v1` | Ask user — may not have Gateway equivalent |
| `cohere.command-r-plus-v1:0` | `cohere/command-r-plus` |

Ask the user to confirm mappings, especially for less common models.

### 4. Migrate `invoke_model` Calls

The `invoke_model` API uses raw JSON bodies specific to each provider. Replace with OpenAI SDK calls.

**Anthropic models via Bedrock:**
```python
# Before
import boto3
import json

bedrock = boto3.client("bedrock-runtime", region_name="us-east-1")

response = bedrock.invoke_model(
    modelId="anthropic.claude-3-5-sonnet-20241022-v2:0",
    body=json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 1024,
        "messages": [{"role": "user", "content": "Hello!"}],
    }),
)
result = json.loads(response["body"].read())
print(result["content"][0]["text"])

# After
from openai import OpenAI
import os

client = OpenAI(
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
)

response = client.chat.completions.create(
    model="anthropic/claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello!"}],
)
print(response.choices[0].message.content)
```

**Meta Llama models via Bedrock:**
```python
# Before
response = bedrock.invoke_model(
    modelId="meta.llama3-1-70b-instruct-v1:0",
    body=json.dumps({
        "prompt": "<s>[INST] Hello! [/INST]",
        "max_gen_len": 512,
        "temperature": 0.7,
    }),
)
result = json.loads(response["body"].read())
print(result["generation"])

# After
response = client.chat.completions.create(
    model="meta-llama/llama-3.1-70b-instruct",
    max_tokens=512,
    temperature=0.7,
    messages=[{"role": "user", "content": "Hello!"}],
)
print(response.choices[0].message.content)
```

### 5. Migrate `converse` / `converse_stream` Calls

The Bedrock Converse API has its own message format. Convert to OpenAI format.

```python
# Before
response = bedrock.converse(
    modelId="anthropic.claude-3-5-sonnet-20241022-v2:0",
    messages=[
        {
            "role": "user",
            "content": [{"text": "Hello!"}],
        }
    ],
    inferenceConfig={"maxTokens": 1024, "temperature": 0.7},
)
print(response["output"]["message"]["content"][0]["text"])

# After
response = client.chat.completions.create(
    model="anthropic/claude-sonnet-4-20250514",
    max_tokens=1024,
    temperature=0.7,
    messages=[{"role": "user", "content": "Hello!"}],
)
print(response.choices[0].message.content)
```

**Message format mapping:**
- Bedrock `{"content": [{"text": "..."}]}` → OpenAI `{"content": "..."}`
- Bedrock `inferenceConfig.maxTokens` → OpenAI `max_tokens`
- Bedrock `inferenceConfig.temperature` → OpenAI `temperature`
- Bedrock `inferenceConfig.topP` → OpenAI `top_p`
- Bedrock `system` → OpenAI `messages` with `role: "system"`

### 6. Migrate Streaming

```python
# Before
response = bedrock.invoke_model_with_response_stream(
    modelId="anthropic.claude-3-5-sonnet-20241022-v2:0",
    body=json.dumps({...}),
)
for event in response["body"]:
    chunk = json.loads(event["chunk"]["bytes"])
    if chunk["type"] == "content_block_delta":
        print(chunk["delta"]["text"], end="")

# After
stream = client.chat.completions.create(
    model="anthropic/claude-sonnet-4-20250514",
    messages=[{"role": "user", "content": "Hello!"}],
    stream=True,
)
for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

```python
# Before (converse_stream)
response = bedrock.converse_stream(
    modelId="anthropic.claude-3-5-sonnet-20241022-v2:0",
    messages=[...],
)
for event in response["stream"]:
    if "contentBlockDelta" in event:
        print(event["contentBlockDelta"]["delta"]["text"], end="")

# After (same as above)
stream = client.chat.completions.create(
    model="anthropic/claude-sonnet-4-20250514",
    messages=[{"role": "user", "content": "Hello!"}],
    stream=True,
)
for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### 7. Install OpenAI SDK

If not already present, add `openai` to the project's dependencies:
- Python: add to `requirements.txt` or `pyproject.toml`
- Inform the user to run `pip install openai` or `poetry add openai`

### 8. Clean Up AWS Dependencies

Ask the user if `boto3` is used for anything other than Bedrock. If not:
- It can be removed from dependencies
- AWS credential env vars can be removed

If `boto3` is used for other AWS services, only remove Bedrock-specific code.

Comment out (do NOT delete) old env vars:
```
# AWS_ACCESS_KEY_ID=...            # No longer needed for Bedrock — using MERGE_GATEWAY_API_KEY
# AWS_SECRET_ACCESS_KEY=...        # No longer needed for Bedrock — using MERGE_GATEWAY_API_KEY
MERGE_GATEWAY_API_KEY=mg_your_api_key_here
MERGE_GATEWAY_BASE_URL=https://gateway.merge.dev
```

### 9. Verify

Generate a test script:

```python
import os
from openai import OpenAI

client = OpenAI(
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
)

response = client.chat.completions.create(
    model="openai/gpt-4o",
    messages=[{"role": "user", "content": "Say 'Bedrock migration successful!' and nothing else."}],
)
print(response.choices[0].message.content)
```

## Cross-Cutting Rules

- **Never delete old configuration** — comment out old env vars with a note about the replacement.
- **Idempotency** — Check if migration is already partially applied before making changes.
- **Provider-prefixed models** — ALL model names must use `provider/model` format.
- **OpenAI SDK base URL** — Always append `/v1`: `os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1"`.
- **Ask before removing boto3** — It may be used for other AWS services beyond Bedrock.
