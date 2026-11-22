# @pipeworx/wv-sos-business

West Virginia Secretary of State business organization records: search organisations by name and read one by its organization id, live per request.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2034+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `wv_search_businesses(name, active_only?, limit?)` — organisations whose name contains the query: organization id (`file_number`), name, entity type, locale (domestic = formed in West Virginia, foreign = formed elsewhere and registered there), profit class, status, effective / established / termination dates and termination reason. Each row is graded `exact`, `legal_form_variant` or `name_prefix`; `hits` counts only the first two. `total_in_register` is the number of organisation rows the register returned. Fictitious-name, trademark and emblem registrations are marked in `name_kind`; they are not companies. A search takes roughly 3 seconds, because the register's form has to be loaded and then submitted after a short pause, as a person at a browser would.
- `wv_get_business(org_id)` — one organisation by organization id, e.g. `253587`: the same fields plus business purpose (industry code and text), charter county, charter state and control number.

## Auth

Keyless.

## Data sources

- West Virginia Secretary of State, business organization records. Every response carries `source` and `data_as_of` (the date of the live read).

The register publishes no status column. `status_normalized` is `active` when no termination date or reason is recorded and `inactive` otherwise; `status` says which.

Entity-level only. No individuals' names are served (officers, members, organisers and the notice-of-process agent are not returned) and no addresses. Dissolved and revoked organisations are included unless `active_only` is set. Trade-name (DBA) and name-change tables on the register are not searched.

A bounce, an HTTP error or a page that is not the search form is reported as an error, never as an empty result.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "wv-sos-business": {
      "url": "https://gateway.pipeworx.io/wv-sos-business/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/wv-sos-business/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2034+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/wv_search_businesses \
  -H 'Content-Type: application/json' \
  -d '{"name":"Acme Alternator, LLC","limit":3}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/wv_search_businesses`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-wv-sos-business` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/wv-sos-business/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "wv-sos-business": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-wv-sos-business"]
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
ask_pipeworx({ question: "your question about Wv Sos Business data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
