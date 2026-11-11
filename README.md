# Conduit Router — Governed MCP Routing

[![mcptry](https://mcptry.com/badge/org.aimlds/conduit-router)](https://mcptry.com/servers/org.aimlds/conduit-router)

**The public listing face of the Conduit Router.** The Router's source is
proprietary and lives in a private repository; this repo is its public
home for directory listings — the same pattern as
`nrupala/conduit-filings-fundamentals` for the filings connector.

Governed MCP routing: every agent call spend-capped, metered, sealed
with a verifiable receipt.

- **Endpoint:** `https://router.pipavia.com/mcp` (streamable HTTP)
- **Discovery is open:** any client may call `initialize`, `ping`, and
  `tools/list` without a credential and evaluate the server first.
- **Execution is keyed:** every tool call requires a Conduit key.
  Get one at [pipavia.com](https://pipavia.com).
- **Registry:** `org.aimlds/conduit-router` on the
  [Official MCP Registry](https://registry.modelcontextprotocol.io)
  — the `server.json` in this repo is the published record.
- **Release log:** [pipavia.com/releases](https://pipavia.com/releases)
- **Verify a receipt:** [router.pipavia.com/verify](https://router.pipavia.com/verify)

Tools (6): `route_chat` · `list_models` · `get_usage` · `get_receipt` ·
`verify_receipt` · `get_policy` — each carries a title, a plain
description, a described argument schema, and read/write annotations.

Sold by 2351485 ALBERTA INC. (AIMLDS) · Contact: info@pipavia.com

Copyright © 2026 2351485 ALBERTA INC.
