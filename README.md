# Captapi MCP server

Captapi's hosted [Model Context Protocol](https://modelcontextprotocol.io) server gives Claude, Cursor, VS Code and other MCP clients read access to public social media data through one API key. Each Captapi REST endpoint is exposed as one MCP tool that returns JSON.

This repository holds documentation and the registry manifest only. It contains no server source code.

- **Endpoint:** `https://api.captapi.com/mcp`
- **Transport:** Streamable HTTP (stateless, JSON-RPC over `POST`)
- **Auth:** `Authorization: Bearer <key>` or `x-api-key: <key>`
- **Official MCP Registry:** [`io.github.CDCStream/captapi`](https://registry.modelcontextprotocol.io/v0.1/servers?search=io.github.CDCStream/captapi)
- **Docs:** [captapi.com/docs/integrations](https://captapi.com/docs/integrations)

## What the tools cover

Public data only: nothing behind a login. For LinkedIn, only pages visible without signing in.

As of October 2026 the server lists 178 tools. Tool names follow `<platform>_<action>`, for example `youtube_transcript`, `tiktok_comments` or `instagram_profile`.

**Platforms:** TikTok, YouTube, Facebook, Instagram, GitHub, LinkedIn, Reddit, X (Twitter), Spotify, Threads, Rumble, Pinterest, Twitch, Google Maps, Truth Social, Kwai, SoundCloud, Bluesky, Snapchat, Kick, Amazon shop pages, and link-in-bio pages (Linktree, Linkme, Linkbio, Pillar, Komi).

**Tool categories:**

- Profiles, channels and pages
- Posts, videos and their details
- Comments and comment replies
- Transcripts and AI summaries
- Search (keywords, hashtags, users)
- Ad libraries: Meta (Facebook), TikTok, Google and LinkedIn
- Cross-platform post analytics (`analytics_post`, `analytics_compare`)
- Google Maps business search
- Your own Captapi account: balance, daily usage, request history (`account_*`)

Run `tools/list` against the endpoint for the current list with exact input schemas. Listing tools does not need a key.

## Get a key

Sign up at [captapi.com](https://captapi.com/signup), then create a key at [captapi.com/dashboard/api-keys](https://captapi.com/dashboard/api-keys). New accounts get 100 free credits after email verification; no credit card is needed.

Each tool call uses the credits of the matching endpoint, listed on its page under [captapi.com/apis](https://captapi.com/apis). Calls that fail are not charged.

Keep your key out of shared config files and version control.

## Setup

Replace `capt_live_xxxxxxxxxxxxxxxx` with your key.

### Claude Code

```bash
claude mcp add --transport http captapi https://api.captapi.com/mcp \
  --header "Authorization: Bearer capt_live_xxxxxxxxxxxxxxxx"
```

### Cursor

`~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (per project):

```json
{
  "mcpServers": {
    "captapi": {
      "url": "https://api.captapi.com/mcp",
      "headers": {
        "Authorization": "Bearer capt_live_xxxxxxxxxxxxxxxx"
      }
    }
  }
}
```

### Claude Desktop

Claude Desktop's config file starts local (stdio) servers, so use the [`@captapi/mcp`](https://www.npmjs.com/package/@captapi/mcp) npm package, which calls the same API. Open Settings → Developer → Edit Config and add:

```json
{
  "mcpServers": {
    "captapi": {
      "command": "npx",
      "args": ["-y", "@captapi/mcp"],
      "env": {
        "CAPTAPI_API_KEY": "capt_live_xxxxxxxxxxxxxxxx"
      }
    }
  }
}
```

Requires Node.js. Restart Claude Desktop after saving.

### Any MCP client (Streamable HTTP)

Point the client at `https://api.captapi.com/mcp` and send the key in a header. Either form works:

```
Authorization: Bearer capt_live_xxxxxxxxxxxxxxxx
x-api-key: capt_live_xxxxxxxxxxxxxxxx
```

Raw check with curl:

```bash
curl -s https://api.captapi.com/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Authorization: Bearer capt_live_xxxxxxxxxxxxxxxx" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"account_balance","arguments":{}}}'
```

The server is stateless: no session ID is needed, and `GET` on the endpoint returns 405. Without a key, `initialize` and `tools/list` work, and `tools/call` returns an error result asking for a key.

## Registry manifest

[`server.json`](server.json) is a copy of the manifest published to the official MCP Registry as `io.github.CDCStream/captapi`.

## Security

See [SECURITY.md](SECURITY.md).

## License

The documentation and configuration in this repository are MIT licensed. See [LICENSE](LICENSE). Use of the Captapi API is governed by the [Terms of Service](https://captapi.com/legal/terms) and [Acceptable Use Policy](https://captapi.com/legal/acceptable-use).
