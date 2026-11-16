# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted compliance-evidence MCP server. Labelled subset: 2 of the 4 captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `ground_claim`, `verify_pack`. The capture carries no `title` keys, so none are served.
- Omitted (SAFETY): `screen_entity` (screens a caller-named entity against OFAC/LEIE/SAM exclusion and sanctions lists) and `build_evidence_pack` (the same named-entity screening is one of its integral legs and cannot be separated from the tool). Neither is registered, served, or offered as a CLI command.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `claimproof error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `COMPLIANCEEVIDENCE_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.
