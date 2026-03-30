
# Build a Tool-Use Agent

Scaffold a function-calling agent loop using the Merge Gateway SDK.

## Language Support

The Merge Gateway SDK is available in both **Python** and **TypeScript/Node**:

- **Python:** `pip install merge-gateway-sdk`
- **TypeScript/Node:** `npm install merge-gateway-sdk`

Detect the user's stack and scaffold the agent in the appropriate language.

## Steps

### 1. Gather Requirements

If the user wants a quick start, skip the questions and scaffold with these defaults:
- **Agent purpose:** General-purpose assistant
- **Tools:** One example tool (stub with TODO)
- **Model:** `openai/gpt-4o`
- **System prompt:** `"You are a helpful assistant."`

Otherwise, ask the user:
- **Agent purpose** — What should the agent do? (e.g., "research assistant", "data analyst", "customer support bot")
- **Tools needed** — What functions should the agent be able to call? (e.g., "search the web", "query a database", "read files")
- **Model preference** — Default: `openai/gpt-4o`. Other options: `anthropic/claude-sonnet-4-20250514`, `openai/gpt-4o-mini`

### 2. Detect Language

Determine Python or TypeScript from the project:
- Python: `pyproject.toml`, `requirements.txt`, `setup.py`, `*.py` files
- TypeScript: `package.json`, `tsconfig.json`, `*.ts` files

If both are present, ask the user which they prefer.

### 3. Install the Merge Gateway SDK

Python:
```bash
pip install merge-gateway-sdk
```

TypeScript/Node:
```bash
npm install merge-gateway-sdk
```

### 4. Scaffold the Agent

Create the agent file with these components:

**Python agent (`agent.py`):**

```python
import json
import os
from merge_gateway import MergeGateway

client = MergeGateway(
    api_key=os.environ["MERGE_GATEWAY_API_KEY"],
    base_url=os.environ["MERGE_GATEWAY_BASE_URL"] + "/v1",
)

# --- Tool definitions ---

tools = [
    {
        "type": "function",
        "name": "tool_name",
        "description": "What this tool does",
        "parameters": {
            "type": "object",
            "properties": {
                "param1": {"type": "string", "description": "Description of param1"},
            },
            "required": ["param1"],
        },
    },
]

# --- Tool implementations ---

def tool_name(param1: str) -> dict:
    """Implement the tool logic here."""
    return {"result": f"Processed: {param1}"}

TOOL_REGISTRY = {
    "tool_name": tool_name,
}

def execute_tool(name: str, arguments: dict) -> str:
    """Execute a tool by name and return the result as a JSON string."""
    func = TOOL_REGISTRY.get(name)
    if not func:
        return json.dumps({"error": f"Unknown tool: {name}"})
    try:
        result = func(**arguments)
        return json.dumps(result)
    except Exception as e:
        return json.dumps({"error": str(e)})

# --- Agent loop ---

def run_agent(user_message: str, system_prompt: str = "You are a helpful assistant.") -> str:
    input_messages = [
        {"type": "message", "role": "system", "content": system_prompt},
        {"type": "message", "role": "user", "content": user_message},
    ]

    while True:
        response = client.responses.create(
            model="openai/gpt-4o",
            input=input_messages,
            tools=tools,
        )
        assistant_msg = response.output[0]

        # Check if the model wants to call a tool
        tool_calls = [block for block in assistant_msg.content if block.type == "tool_use"]

        if not tool_calls:
            # No tool calls — extract text response
            text_blocks = [block for block in assistant_msg.content if block.type == "text"]
            return text_blocks[0].text if text_blocks else ""

        # Process each tool call and send results back
        for tool_call in tool_calls:
            result = execute_tool(tool_call.name, tool_call.input or {})
            input_messages.append({
                "type": "tool_result",
                "tool_use_id": tool_call.id,
                "content": result,
            })

if __name__ == "__main__":
    response = run_agent("Your test prompt here")
    print(response)
```

**TypeScript agent (`agent.ts`):**

```typescript
import { MergeGateway } from "merge-gateway-sdk";

const client = new MergeGateway({
  apiKey: process.env.MERGE_GATEWAY_API_KEY!,
  baseUrl: process.env.MERGE_GATEWAY_BASE_URL + "/v1",
});

// --- Tool definitions ---

const tools = [
  {
    type: "function" as const,
    name: "tool_name",
    description: "What this tool does",
    parameters: {
      type: "object",
      properties: {
        param1: { type: "string", description: "Description of param1" },
      },
      required: ["param1"],
    },
  },
];

// --- Tool implementations ---

function toolName(args: { param1: string }): Record<string, unknown> {
  return { result: `Processed: ${args.param1}` };
}

const TOOL_REGISTRY: Record<string, (args: Record<string, unknown>) => Record<string, unknown>> = {
  tool_name: toolName,
};

function executeTool(name: string, args: Record<string, unknown>): string {
  const func = TOOL_REGISTRY[name];
  if (!func) return JSON.stringify({ error: `Unknown tool: ${name}` });
  try {
    return JSON.stringify(func(args));
  } catch (e) {
    return JSON.stringify({ error: String(e) });
  }
}

// --- Agent loop ---

async function runAgent(userMessage: string, systemPrompt = "You are a helpful assistant."): Promise<string> {
  const inputMessages: any[] = [
    { type: "message", role: "system", content: systemPrompt },
    { type: "message", role: "user", content: userMessage },
  ];

  while (true) {
    const response = await client.responses.create({
      model: "openai/gpt-4o",
      input: inputMessages,
      tools,
    });
    const assistantMsg = response.output[0];

    // Check if the model wants to call a tool
    const toolCalls = assistantMsg.content.filter((block: any) => block.type === "tool_use");

    if (toolCalls.length === 0) {
      // No tool calls — extract text response
      const textBlocks = assistantMsg.content.filter((block: any) => block.type === "text");
      return textBlocks[0]?.text ?? "";
    }

    // Process each tool call and send results back
    for (const toolCall of toolCalls) {
      const result = executeTool(toolCall.name, toolCall.input ?? {});
      inputMessages.push({
        type: "tool_result",
        tool_use_id: toolCall.id,
        content: result,
      });
    }
  }
}

// --- Main ---

runAgent("Your test prompt here").then(console.log);
```

### 5. Customize for the User's Requirements

Based on the user's answers from step 1:
- Replace placeholder tool definitions with real tools matching their requirements
- Write meaningful tool implementations (or stubs with TODO comments if external APIs are needed)
- Set an appropriate system prompt for the agent's purpose
- Use the user's preferred model

### 6. Verify

Offer to run the agent with a test prompt to confirm it works through Gateway.

## Gateway Advantages to Highlight

When explaining the setup to the user, mention:
- **Model swapping** — Change the model string to route to any provider (e.g., switch from `openai/gpt-4o` to `anthropic/claude-sonnet-4-20250514`) without code changes
- **Routing policies** — Configure fallbacks, load balancing, and cost optimization in the Gateway dashboard
- **Unified billing** — One API key, one bill, regardless of which providers the agent uses

## Cross-Cutting Rules

- **Provider-prefixed models** — ALL model names must use `provider/model` format (e.g., `openai/gpt-4o`, not `gpt-4o`).
- **Env vars** — Always use `MERGE_GATEWAY_API_KEY` and `MERGE_GATEWAY_BASE_URL` environment variables, never hardcoded values.
- **Base URL** — The env var `MERGE_GATEWAY_BASE_URL` should be set **without** `/v1` (e.g., `https://api-gateway.merge.dev`). Always append `/v1` in code. If the env var already contains `/v1`, do NOT append it again — check for this to avoid a double `/v1` path.
