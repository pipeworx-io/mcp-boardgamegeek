# @pipeworx/boardgamegeek

Board-game metadata, community ratings and ranks, name search and the trending list from BoardGameGeek's board game database.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1869+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `bgg_get_games(ids, include_stats?)` — up to 20 games by BoardGameGeek id: names, year, player count, play time, categories, mechanics, designers, publishers, and community statistics (rating, Bayes average, ranks, weight, owner counts).
- `bgg_search_games(query, exact?, include_expansions?, limit?)` — find BoardGameGeek ids by name. Reports `total` and `truncated`.
- `bgg_hot_games(limit?)` — the current "hot" list in rank order.
- `bgg_user_collection(username, owned_only?, limit?)` — a member's owned/wishlist games with their own ratings and play counts. Large collections are queued by BoardGameGeek and may need a retry.

## Auth

BYO only. BoardGameGeek answers unauthenticated requests with 401 and issues a token only to a registered application. Pass yours as `_apiKey`; without it every tool refuses with "requires an API key" and makes no upstream call.

**Terms.** BoardGameGeek licenses its API data for strictly non-commercial use, bars using it to train AI/LLM systems, and requires a separate commercial licence (offered case by case) for commercial use; attribution ("Powered by BGG") applies to public-facing uses. Those terms bind whoever registers the application, which is why this pack carries no Pipeworx token. (Quoted from the terms as mirrored on BoardGameGeek's sister-site wikis; the primary pages were behind a bot wall when this was written, so check them before relying on the wording.)

## Data sources

- BoardGameGeek, board game database — game records, community ratings and ranks, member collections.

Notes for the next person: BGG double-encodes description text (`&amp;#10;`), so descriptions are decoded twice. `Not Ranked` is the literal rank value for an unranked game and is returned as `null`. The collection endpoint answers 202 while it builds the list; the pack retries twice. Test fixtures are synthetic documents in BGG's documented shape (no token was available at build); the first keyed call after a token exists is the real shape check.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "boardgamegeek": {
      "url": "https://gateway.pipeworx.io/boardgamegeek/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/boardgamegeek/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1869+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/bgg_get_games`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-boardgamegeek` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/boardgamegeek/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "boardgamegeek": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-boardgamegeek"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Boardgamegeek data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
