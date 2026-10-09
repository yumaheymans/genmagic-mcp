# GenMagic MCP Server

**One [Model Context Protocol](https://modelcontextprotocol.io) server that lets any AI agent generate text, images, speech, music, and video, billed against a single prepaid credit balance.**

GenMagic is one studio and one API for every generative AI model, in every medium. This repository is the connection guide and registry descriptor for its **remote MCP server**: point any MCP client at one URL, drop in your key, and your agent gets six generation tools plus `list_models`: it can pick any of GenMagic's 450+ models (GPT, Claude, Gemini, Veo, Kling, Seedance, FLUX, Recraft and more) by id, or leave the choice to GenMagic.

- **Endpoint:** `https://genmagic.co/api/mcp`
- **Transport:** Streamable HTTP (stateless JSON-RPC)
- **Version:** 1.1.0
- **Auth:** `Authorization: Bearer gm_live_...` (create a key at <https://genmagic.co/get-api-key>)
- **Hosted:** nothing to install or run. It is a live remote server.
- **Registry:** listed on the [Official MCP Registry](https://registry.modelcontextprotocol.io) as `io.github.yumaheymans/genmagic` (see [`server.json`](./server.json)).

## Tools

| Tool | What it does |
| --- | --- |
| `list_models` | The live catalog, newest first: each model's id, price and key details (context window and capabilities for text, voices for speech, resolutions, clip lengths and sound for video). Filter by `category`, `search` or `capability`. No key needed. |
| `generate_text` | Generate text, code, an SVG graphic, or a full web page (optional `type`, optional `model`). |
| `generate_image` | Generate or edit an image; `type=logo` for a brand mark; optional `model`. |
| `generate_speech` | Text to speech, voiced verbatim; optional `model` and `voice`. |
| `generate_music` | An original music track from a description; optional `model`. |
| `create_video` | Start an async video job with any text-to-video model, `duration`, `resolution` and `generate_audio` it supports (returns a job id, charges nothing yet). |
| `get_video` | Poll a video job; charges once, on completion. |

Every generate tool takes an optional `model` id from `list_models`. A model id that does not run that kind of output is refused with a pointer to `list_models`, never swapped for another model; leave `model` out and GenMagic picks a well-priced default.

`initialize`, `ping`, `tools/list` and `list_models` are open; the generate tools require your key. Every tool result self-describes its spend under the MCP `_meta` key `genmagic.ai/usage` (`model` that ran, `cost_cents`, `credits_remaining`, and a `rate` object), so an agent paces itself and tracks its budget from the result it already has.

## Connect

Create a key at <https://genmagic.co/get-api-key> (sign up free and verify your email), then add the server to your client.

### Claude Code

```bash
claude mcp add --transport http genmagic https://genmagic.co/api/mcp \
  --header "Authorization: Bearer gm_live_your_key"
```

### Cursor, VS Code, Windsurf, and most clients

Add a `genmagic` entry to your MCP config (`.cursor/mcp.json`, `.vscode/mcp.json`, or the client's equivalent):

```json
{
  "mcpServers": {
    "genmagic": {
      "url": "https://genmagic.co/api/mcp",
      "headers": { "Authorization": "Bearer gm_live_your_key" }
    }
  }
}
```

### Claude Desktop

Add GenMagic as a custom connector in **Settings → Connectors**, using the URL `https://genmagic.co/api/mcp` and a header `Authorization: Bearer gm_live_your_key`.

### Any JSON-RPC client (raw)

The server is plain JSON-RPC over HTTP, so you can drive it with `curl`:

```bash
# List the tools (no key required)
curl https://genmagic.co/api/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{ "jsonrpc": "2.0", "id": 1, "method": "tools/list" }'

# Generate an image (key required)
curl https://genmagic.co/api/mcp \
  -H "Authorization: Bearer gm_live_your_key" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0", "id": 2, "method": "tools/call",
    "params": {
      "name": "generate_image",
      "arguments": { "prompt": "a cobalt prism refracting into a spectrum, dark studio, 3D render", "size": "1024x1024" }
    }
  }'
```

## How billing works

There is no separate agent meter. Every generation, from the studio, the REST API, or this MCP server, draws the underlying model's real provider cost from one prepaid USD balance (1 credit = 1 US cent). A new account gets a one-time $0.50 starter balance after email verification, one per person; top up in $5 to $100 increments, or subscribe for a renewing balance with a subscriber bonus and commercial-use rights. `create_video` and a still-rendering `get_video` poll charge nothing; the clip's cost lands on the poll that completes it. See <https://genmagic.co/pricing>.

## On-brand generation

Turn on **personalization** once in the app (connect a website or LinkedIn) and every tool call comes out shaped by your brand's voice, audience, and colors, no prompt changes required. Visual tool results additionally report a `brand_fit` read-back so an agent can regenerate anything that drifted off-palette.

## Links

- Studio: <https://genmagic.co> · API key: <https://genmagic.co/get-api-key>
- Full API + MCP docs: <https://genmagic.co/developers>
- Machine-readable discovery: [`/.well-known/mcp.json`](https://genmagic.co/.well-known/mcp.json) · [`/llms.txt`](https://genmagic.co/llms.txt) · [`/openapi.json`](https://genmagic.co/openapi.json)
- Pricing: <https://genmagic.co/pricing>

## License

MIT. See [LICENSE](./LICENSE). This repository is connection documentation and the registry descriptor; the GenMagic service itself is operated at genmagic.co.
