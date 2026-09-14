---
description: Connect Claude Code (or any coding agent) to a company's Merge for Workforce deployment with the `mfw` CLI — company models through AI Gateway and provisioned tools over MCP. Use when the user mentions Merge for Workforce, Workforce, `mfw`, "our company's models/tools through Merge", an employee Gateway credential, or asks to point Claude Code / Codex / Cursor at Merge. Prefer this over hand-writing env vars or MCP server entries.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

# Set up Merge for Workforce with the `mfw` CLI

An employee whose company runs Merge for Workforce reaches two things through Merge: **models** (through AI Gateway, with the company's routing policy, budget and logs) and **tools** (their provisioned connectors, over one MCP endpoint). The `mfw` CLI does both halves from the terminal. It signs the employee in through the company identity provider, mints a per-device Gateway credential, and emits the exact env vars and MCP config the AI tool needs.

**Do not** send the user to create a Gateway API key in a dashboard, ask them for a provider key, or hand-write `ANTHROPIC_BASE_URL` / `.mcp.json` entries for Merge. Those paths bypass the employee credential, so the request is not attributed to them and their company's policy does not apply. Use the CLI.

## Steps

### 0. Check for plugin updates

```
claude plugin update merge-gateway@merge-gateway-skills
```

**Run this update and wait for it to complete before continuing to Step 1.**

### 1. Is the desktop client already doing this?

If the company deployed the **Workforce desktop client** through its MDM, the machine is already configured: MCP tools, Gateway base URL and skills are written for the employee, and nothing below is needed. Check with `/status` in Claude Code (auth source `ANTHROPIC_AUTH_TOKEN`, base URL ending in `/v1/anthropic`) and `/mcp` (a Merge server listed). If both show, stop here.

### 2. Install the CLI

```bash
pipx install merge-workforce     # recommended: isolated env, puts `mfw` on PATH
# or
pip install merge-workforce
```

Confirm with `mfw --help`. The command is `mfw`; the PyPI package is `merge-workforce`.

### 3. Sign in

```bash
mfw login --gateway-url https://ah-api.merge.dev
```

This opens the browser for the company's SSO and stores an access token. The URL is the Workforce backend; `https://ah-api.merge.dev` is the production default and is remembered for later commands. If the company runs a dedicated environment, use the URL their admin gave them.

### 4. Provision this machine's Gateway credential

```bash
mfw setup
```

Mints a per-device Gateway credential for the employee and stores it under `~/.mfw/` (0600). Every machine gets its own credential, so spend is attributed to the employee and one laptop can be revoked without touching the others. `mfw setup` prints the next commands.

### 5. Models: route the AI tool through the Gateway

In the shell that launches the tool:

```bash
eval "$(mfw env)"
claude
```

`mfw env` prints `export` lines for the credential under every spelling AI tools read: `ANTHROPIC_BASE_URL` / `ANTHROPIC_AUTH_TOKEN` (Claude Code), `OPENAI_BASE_URL` / `OPENAI_API_KEY` (Codex and OpenAI-SDK tools), and `MERGE_GATEWAY_*` (the Gateway SDKs). It also sets `ANTHROPIC_API_KEY` to empty on purpose: Claude Code prefers that variable over the auth token when both are set, and would bypass the Gateway.

For one process instead of the whole shell:

```bash
mfw run -- claude
```

To make it stick, add `eval "$(mfw env)"` to `~/.zshrc` or `~/.bashrc`. Do not paste the printed values into a config file; the credential rotates and `mfw env` always reflects the current one.

`mfw env` also turns on Claude Code's trace propagation (so a prompt's model requests and tool calls share one trace id across the Gateway request log and the tool-call log) and, from CLI 0.1.3, `ENABLE_TOOL_SEARCH=true` so Claude Code loads MCP tools on demand. Variables the user already exports are left alone; `--no-trace` skips the trace block.

### 6. Tools: hand the provisioned connectors to the AI tool

```bash
mfw mcp --write .mcp.json          # in the project root; Claude Code picks it up
```

This writes an `mcpServers` entry for the company's MCP endpoint with a short-lived token. The agent calls tools **as the employee**, exactly as the console governs them. Re-run when the token lapses (the tool reports an OAuth/401 error).

If the employee is provisioned for many connectors the tool list can run to thousands of tools. Scope it (CLI 0.1.3+):

```bash
mfw mcp --connectors notion,github --write .mcp.json
```

For other MCP clients (Cursor, an SDK agent) print the config with `mfw mcp` and paste it where that client reads it.

### 7. Verify

- `mfw models` lists the models the employee's group allows; `mfw ask -m <model> "hi"` makes one real call.
- In Claude Code, `/status` shows auth source `ANTHROPIC_AUTH_TOKEN` and the Gateway base URL; `/mcp` lists the Merge server and its tools.
- `mfw chat -m <model> "<prompt>"` runs a built-in agent with the employee's tools and needs no other client; useful to prove both halves work.

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| `mfw setup` says Merge Gateway is not set up for the organization | The login resolved to the wrong identity (an Agent Handler console session in the same browser). Sign out of that console or use another browser profile, then `mfw login` again. |
| Claude Code ignores the Gateway | `ANTHROPIC_API_KEY` is set to a non-empty value somewhere and wins. `mfw env` blanks it; check `~/.zshrc`, `.claude/settings*.json`, and `/logout` a prior Anthropic login. |
| Model picker shows only Anthropic models | Claude Code skips the Gateway model list unless `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1`. |
| First request fails with "prompt is too long" | Thousands of MCP tools were loaded up front. Set `ENABLE_TOOL_SEARCH=true` (CLI 0.1.3 `mfw env` does) or scope with `mfw mcp --connectors …`. |
| Tool calls return an OAuth / 401 error | The MCP token in `.mcp.json` lapsed. Re-run `mfw mcp --write .mcp.json`. |
| A tool returns `reauth_required` | The employee's connector needs re-authorizing in the Workforce console; not a CLI problem. |

## Never

- Never print, log, or paste the Gateway credential or the MCP bearer token into files other than what `mfw` writes.
- Never replace the employee credential with a personal provider key or a shared team key.
- Never edit `~/.mfw/`; use `mfw logout` and `mfw login` instead.
