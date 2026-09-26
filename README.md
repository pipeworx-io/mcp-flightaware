# mcp-flightaware

FlightAware MCP — wraps FlightAware AeroAPI v4 (aeroapi.flightaware.com)

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1683+ live data sources.

## Tools

| Tool | Description |
|------|-------------|
| `flightaware_flight_status` | Get real-time flight status for a flight number or tail number: origin/destination airports, scheduled/estimated/actual departure & arrival times, current status, and aircraft type. Returns a flights[] array (one flight ident can map to several dated flights). Example: flightaware_flight_status({ ident: "UAL123", _apiKey: "your-key" }) |
| `flightaware_flight_track` | Get the historical position track (breadcrumb trail of lat/lon/altitude/groundspeed points) for a specific flight. The `id` must be an AeroAPI fa_flight_id (e.g. "UAL123-1234567890-airline-0123") — get one from flightaware_flight_status first (the fa_flight_id field). Example: flightaware_flight_track({ id: "UAL123-1234567890-airline-0123", _apiKey: "your-key" }) |
| `flightaware_airport_flights` | List flights at an airport — arrivals and/or departures with scheduled & actual times. The `id` is an airport code (ICAO like "KSFO" or IATA like "SFO"). Use `type` to pick "arrivals", "departures", or "all" (default "all", which also includes scheduled flights). Example: flightaware_airport_flights({ id: "KSFO", type: "departures", _apiKey: "your-key" }) |
| `flightaware_flight_position` | Get the current/last-known position of an in-air flight: latitude, longitude, altitude, groundspeed, heading and timestamp. The `id` must be an AeroAPI fa_flight_id (from flightaware_flight_status). Example: flightaware_flight_position({ id: "UAL123-1234567890-airline-0123", _apiKey: "your-key" }) |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "flightaware": {
      "url": "https://gateway.pipeworx.io/flightaware/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/flightaware/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1683+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/flightaware_flight_status`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "flightaware": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-flightaware"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-flightaware
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Flightaware data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
