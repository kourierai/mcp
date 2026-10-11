# Kourier MCP Server

Remote [Model Context Protocol (MCP)](https://modelcontextprotocol.io) server for [Kourier](https://kourier.sh) — **flat-rate hosted AI inference for coding agents**.

Use the MCP server to discover plans, check model limits, get API-key instructions, and configure your coding client. Kourier serves DeepSeek V4.1 Flash through OpenAI-compatible and Anthropic-compatible inference APIs, with no per-token billing or request-volume quotas. Plans differ by concurrency.

```text
https://kourier.sh/mcp
```

**Transport:** Streamable HTTP, stateless, JSON responses. **Authentication:** none required for the public MCP tools.

> MCP provides service information and setup instructions, not model inference. Inference uses a subscription API key at `https://api.kourier.sh/v1` (OpenAI-compatible) or `https://api.kourier.sh/anthropic` (Anthropic-compatible).

This repository documents the hosted service; there is no local server to install or run. The live MCP server is currently read-only: it does not purchase subscriptions, issue keys, or manage accounts directly.

## Quick start

### Claude Code

```bash
claude mcp add --transport http kourier https://kourier.sh/mcp
```

### Cursor / clients using `mcpServers`

Add this entry to your client's MCP configuration:

```json
{
  "mcpServers": {
    "kourier": {
      "url": "https://kourier.sh/mcp"
    }
  }
}
```

Configuration formats vary by client; use the remote Streamable HTTP URL above where your client requests a server URL.

Once connected, ask: **“What are Kourier's plans, and how do I configure my coding client to use it?”**

Adding this MCP server does not change the model powering your coding client. Use `get_setup_instructions` to configure Kourier as the inference provider separately.

## Tools

All currently hosted tools are read-only and require no authentication.

| Tool | Arguments | What it does |
| --- | --- | --- |
| `get_pricing` | None | Returns public plans, monthly prices, concurrency limits, and usage-cap information. |
| `get_models` | None | Returns the served model's exact API identifier and context/output limits. |
| `get_api_key_instructions` | None | Explains dashboard signup, agent sign-in, and OAuth with PKCE. Does not return an API key itself. |
| `get_setup_instructions` | Optional `harness` | Returns configuration guidance for `oh-my-pi`, `claude-code`, `codex`, or `openai-compatible`. Omit the argument for generic OpenAI-compatible settings. |

For example, call `get_setup_instructions` with:

```json
{
  "harness": "claude-code"
}
```

The server also exposes the **`kourier-setup`** Markdown resource and the **`configure_kourier`** prompt, which accepts the same optional `harness` value.

## Hosted inference

### Plans

| Plan | Monthly price (USD) | Concurrent requests |
| --- | --- | --- |
| Starter | $20 | 1 |
| Pro | $50 | 3 |

Neither public plan has a five-hour, weekly, or monthly request or token quota. A concurrency slot becomes available when its request finishes. “No usage quotas” does not mean unlimited simultaneous requests.

Consult [`get_pricing`](https://kourier.sh/mcp) or [the pricing page](https://kourier.sh/pricing) for current terms; the MCP endpoint accepts JSON-RPC POST requests, not ordinary browser GET requests.

### Model and endpoints

| Setting | Value |
| --- | --- |
| Model | DeepSeek V4.1 Flash |
| Exact API model identifier | `deepseek-v4.1-flash` |
| OpenAI-compatible base URL | `https://api.kourier.sh/v1` |
| Anthropic-compatible base URL | `https://api.kourier.sh/anthropic` |
| Maximum combined prompt + output context | 262,144 tokens (256K) |
| Maximum output per response | 16,384 tokens |
| Authentication | `Authorization: Bearer <key>`; the Anthropic surface also accepts `x-api-key` |

Use the exact model identifier; aliases are not supported. Check `get_models` and [the model documentation](https://docs.kourier.sh/docs/models) for current availability and limits.

## Getting connected

1. [Create an account](https://app.kourier.sh/signup).
2. Choose Starter or Pro in Billing.
3. Create an API key in Dashboard → API Keys.
4. Call `get_setup_instructions` for your client and apply the returned settings.

For agent-assisted sign-in, follow [Kourier's auth.md](https://kourier.sh/auth.md) and [the agent sign-in guide](https://docs.kourier.sh/docs/agent-sign-in). Human approval happens in the browser; connecting to the public MCP server alone does not authorize billing or grant inference access.

Keep API keys in environment variables or your client's private credential store. Never commit them to a repository or paste them into public issues. The public information tools do not need your inference key.

### Inference request with curl

With an active subscription and `KOURIER_API_KEY` set in your environment:

```bash
curl https://api.kourier.sh/v1/chat/completions \
  -H "Authorization: Bearer $KOURIER_API_KEY" \
  -H "Content-Type: application/json" \
  --data '{
    "model": "deepseek-v4.1-flash",
    "messages": [{"role": "user", "content": "Explain what this coding API supports."}],
    "max_tokens": 256
  }'
```

## Verify the MCP connection

Send an initialization request without an API key:

```bash
curl https://kourier.sh/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  --data '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
      "protocolVersion": "2025-11-25",
      "capabilities": {},
      "clientInfo": {"name": "kourier-quickstart", "version": "1.0.0"}
    }
  }'
```

The response identifies the server as `kourier-info` and advertises tools, resources, and prompts. Supported protocol versions are `2025-11-25`, `2025-06-18`, and `2025-03-26`.

List the available tools:

```bash
curl https://kourier.sh/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  --data '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}'
```

The endpoint does not offer a persistent SSE stream or MCP sessions. `GET` and `DELETE` return `405`; use JSON-RPC `POST`. Normal MCP clients handle protocol initialization automatically.

## Links

- **Website:** [kourier.sh](https://kourier.sh)
- **Pricing:** [kourier.sh/pricing](https://kourier.sh/pricing)
- **Dashboard:** [app.kourier.sh](https://app.kourier.sh)
- **Documentation:** [docs.kourier.sh/docs](https://docs.kourier.sh/docs)
- **Agent sign-in:** [kourier.sh/auth.md](https://kourier.sh/auth.md)
- **MCP server card:** [kourier.sh/.well-known/mcp/server-card.json](https://kourier.sh/.well-known/mcp/server-card.json)
- **Setup skill:** [kourier-setup/SKILL.md](https://kourier.sh/.well-known/agent-skills/kourier-setup/SKILL.md)

For documentation corrections, open an issue in this repository. For account or billing help, use the support options on Kourier's website or dashboard; do not disclose account credentials here.
