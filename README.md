# GenMagic MCP Server

**One [Model Context Protocol](https://modelcontextprotocol.io) server that lets any AI agent generate text, images, speech, music, and video, billed against a single prepaid credit balance.**

GenMagic is one studio and one API for every generative AI model, in every medium. This repository is the connection guide and registry descriptor for its **remote MCP server**: point any MCP client at one URL, drop in your key, and your agent gets six generation tools that route to the best underlying model automatically.

- **Endpoint:** `https://genmagic.co/api/mcp`
- **Transport:** Streamable HTTP (stateless JSON-RPC)
- **Auth:** `Authorization: Bearer gm_live_...` (create a key at <https://genmagic.co/app/settings>)
- **Hosted:** nothing to install or run. It is a live remote server.
- **Registry:** listed on the [Official MCP Registry](https://registry.modelcontextprotocol.io) as `io.github.yumaheymans/genmagic` (see [`server.json`](./server.json)).

## Tools

| Tool | What it does |
| --- | --- |
| `generate_text` | Generate text, code, an SVG graphic, or a full web page (optional `type`). |
| `generate_image` | Generate or edit an image; `type=logo` for a brand mark. |
| `generate_speech` | Text to speech, voiced verbatim. |
| `generate_music` | An original music track from a description. |
| `create_video` | Start an async video job (returns a job id, charges nothing yet). |
| `get_video` | Poll a video job; charges once, on completion. |

`initialize`, `ping`, and `tools/list` are open; `tools/call` requires your key. Every tool result self-describes its spend in the MCP `_meta` field (`cost_cents`, `credits_remaining`, and a `rate` object), so an agent paces itself and tracks its budget from the result it already has.

## Connect

Create a key at <https://genmagic.co/app/settings>, then add the server to your client.

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

There is no separate agent meter. Every generation, from the studio, the REST API, or this MCP server, draws the underlying model's real provider cost from one prepaid USD balance (1 credit = 1 US cent). New accounts get a one-time $0.50 starter balance; top up in $5 to $100 increments, or subscribe for a renewing balance with a subscriber bonus and commercial-use rights. `create_video` and a still-rendering `get_video` poll charge nothing; the clip's cost lands on the poll that completes it. See <https://genmagic.co/pricing>.

## On-brand generation

Turn on **personalization** once in the app (connect a website or LinkedIn) and every tool call comes out shaped by your brand's voice, audience, and colors, no prompt changes required. Visual tool results additionally report a `brand_fit` read-back so an agent can regenerate anything that drifted off-palette.

## Links

- Studio and API keys: <https://genmagic.co>
- Full API + MCP docs: <https://genmagic.co/developers>
- Machine-readable discovery: [`/.well-known/mcp.json`](https://genmagic.co/.well-known/mcp.json) · [`/llms.txt`](https://genmagic.co/llms.txt) · [`/openapi.json`](https://genmagic.co/openapi.json)
- Pricing: <https://genmagic.co/pricing>

## License

MIT. See [LICENSE](./LICENSE). This repository is connection documentation and the registry descriptor; the GenMagic service itself is operated at genmagic.co.
