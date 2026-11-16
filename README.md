# claimproof

Did a public company actually do what its filing says, and is that compliance-evidence pack you were handed still the pack that was issued? claimproof is a local forwarding client for the hosted Dropwatch compliance-evidence MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the service, returning the answers exactly as the service sends them back.

This is a labelled subset build: 2 of the 4 captured tools are served. The two omitted tools both screen caller-named entities against federal exclusion and sanctions lists, and this rebuild does not perform by-name screening (see "Omissions" below). Neither kept tool screens, scores, or looks up any named person or entity against any list.

## The tools

- **`ground_claim`** — Ground one public company's disclosure claim against current public government facts (SEC 10-K filings versus USAspending records). Returns the dual-cited record when a verifiable contradiction exists, or an honest null — "checked, no contradiction at medium/high confidence as of now" — which is itself audit evidence. `ticker` is required.
- **`verify_pack`** — Verify a previously issued evidence pack's hash-chain receipt, proving the artifact is unaltered since it was generated. Pass the pack `id`.

## Omissions

- **`screen_entity`** — OMITTED (SAFETY). It screens a caller-named entity (a name, with optional NPI / UEI identifiers) against OFAC SDN, OFAC Consolidated, OIG LEIE and SAM exclusion/sanctions lists and returns the screen result. Performing a by-name screen is not built here; the tool is not registered, does not appear in `tools/list`, and is not a CLI command.
- **`build_evidence_pack`** — OMITTED (SAFETY, same family). Its own description states it screens each named entity against federal exclusion/sanctions lists as one of the pack's three integral legs, via its `entities` argument. The screening cannot be separated from the tool, so serving the tool would serve the screen; it is omitted on the same terms.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/claimproof mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "claimproof": {
      "command": "npx",
      "args": ["-y", "@reneturcios/claimproof", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/claimproof
docker run --rm ghcr.io/cloudcorprecords/claimproof verify_pack '{"id":"pack-123"}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
claimproof ground_claim '{"ticker":"ACME"}'
claimproof verify_pack '{"id":"pack-123"}'
claimproof --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `COMPLIANCEEVIDENCE_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://evidence.dropwatchhq.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `claimproof error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (the 2 kept tools, deep-equal, in order, with no titles invented), asserts the absence of both omitted tools at the registry, MCP-layer and CLI level, exercises the session handshake and its reuse, verifies that no `Authorization` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
