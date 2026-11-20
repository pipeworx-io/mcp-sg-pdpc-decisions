# @pipeworx/sg-pdpc-decisions

Singapore Personal Data Protection Commission (PDPC) enforcement decisions against organisations under the Personal Data Protection Act: breaches with financial penalties or directions, voluntary undertakings accepted by the Commission, and no-breach findings.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2016+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `sg_pdpc_decisions_search(query?, outcome?, limit?)` -- searches the PDPC enforcement decisions, newest update first. The keyword matches the organisation name and the obligation in the title. `outcome` is `breach`, `voluntary_undertaking`, `no_breach` or `other`. Each hit: decision id, outcome, title, link. `individuals_dropped` counts the decisions about individuals that were not returned. A search with no hit is `found:false` with `empty_reason: no_match`; a source that cannot be read is an error, never an empty result.
- `sg_pdpc_decision_get(id)` -- one decision by its id (such as `breach-of-the-protection-obligation-by-myrepublic`): published title and date, decision type, tags (obligation breached, Financial Penalty, sector), the summary with the penalty amount, and links to the decision documents. An unknown id is `found:false`; a decision about an individual is `found:false` with `empty_reason: not_an_organisation`.

Every response carries `source` and `data_as_of`.

## Auth

Keyless.

## Data sources

- Personal Data Protection Commission Singapore, enforcement decisions. Read live per call, nothing stored.

Privacy: organisations only. Decisions about an individual (a registered salesperson, a financial adviser, a person selling personal data) are dropped and counted. A sole trader trading under a personal name may not be recognisable from the decision title alone; the exclusion works from the title.

Traps worth knowing: search titles are rebuilt from the decision address and can lose punctuation (the decision get returns the published title); `updated` in a search hit is when the PDPC last changed the page, not the decision date; a zero-hit search means no match, not that the organisation has no history with the PDPC.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "sg-pdpc-decisions": {
      "url": "https://gateway.pipeworx.io/sg-pdpc-decisions/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/sg-pdpc-decisions/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2016+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/sg_pdpc_decisions_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"protection obligation","outcome":"breach"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/sg_pdpc_decisions_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-sg-pdpc-decisions` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/sg-pdpc-decisions/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "sg-pdpc-decisions": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-sg-pdpc-decisions"]
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
ask_pipeworx({ question: "your question about Sg Pdpc Decisions data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
