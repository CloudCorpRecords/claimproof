# BUILD REPORT — claimproof

Clean-room rebuild of `com.dropwatchhq/compliance-evidence`
v1.0.0 (remote-only MCP server at
`https://evidence.dropwatchhq.com/mcp`), built 2026-10-28
as a forwarding client. Package: `@reneturcios/claimproof`
0.1.0, registry name `io.github.CloudCorpRecords/claimproof`.

## Surface built

Labelled subset: **2 of 4 captured tools built**, 2 omitted
(see below).

| Tool | Required fields as served |
|------|---------------------------|
| `ground_claim` | `ticker` |
| `verify_pack` | `id` |

Each served tool carries exactly the keys `name`,
`description`, `inputSchema`, exactly as captured: the
capture carries NO `title` keys, so none are served. The
tool literals were generated programmatically from
`harness/fixtures/complianceevidence-tools-original.json`
by the script at
`/tmp/b46-claimproof-scratch/gen_forward.py` (parse +
re-serialise into `src/forward.ts` via a template with a
placeholder, never retyped) — and a post-generation check
(the registration-integrity test suite, plus the parity
harness) confirmed the served literals deep-equal the
fixture's 2 tool objects, in order. Tools are registered
via the MCP SDK low-level Server API
(`ListToolsRequestSchema` / `CallToolRequestSchema`
handlers), not McpServer+zod.

## Omissions — `screen_entity`, `build_evidence_pack`

- OMITTED: `screen_entity` — SAFETY. It screens a
  caller-NAMED entity (a name, with optional NPI / UEI
  identifiers) against OFAC SDN, OFAC Consolidated, OIG
  LEIE and SAM exclusion/sanctions lists and returns the
  screen result. That is screening/searching a
  caller-named subject — the bizfile-omissions /
  eu-sanctions-preflight family. The standing distinction
  governs: returning a published list as a dataset the
  caller filters locally is buildable; performing the
  by-name screen is not built. The NPI identifier class
  covers individual healthcare providers, so named
  individuals are in scope of the tool.
- OMITTED: `build_evidence_pack` — SAFETY, same family.
  Its own description states it "screens each named
  entity against federal exclusion/sanctions
  (exclude-feed: OFAC/LEIE/SAM)" as one of the pack's
  three integral legs, via its `entities` argument
  ([{name, npi?, uei?, state?}]). The screening cannot be
  separated from the tool; serving the tool would serve
  the screen.
- The kept tools screen nothing: `ground_claim` grounds
  one PUBLIC company's disclosure claim against current
  public gov facts (SEC 10-K vs USAspending) and returns
  the cited record or an honest null — a published-data
  read about a public company; `verify_pack` verifies a
  previously issued pack's hash-chain receipt — an
  artifact-verification read.
- Neither omitted tool is registered, appears in
  tools/list, or is a CLI command.
- Absence assertions in `test/forward.test.ts`:
  - Registry level: `TOOL_NAMES` includes neither omitted
    name, no `TOOL_DEFS` entry carries either name,
    `TOOL_DEFS.length === 2`, and `isKnownTool` returns
    false for both.
  - MCP layer: `handleCallTool` rejects each omitted tool
    with McpError MethodNotFound before any network hop
    (upstream call count 0).
  - CLI level: `claimproof screen_entity '{}'` and
    `claimproof build_evidence_pack '{}'` each print
    Usage on stderr and exit 2, and the usage tool list
    names neither omitted tool.

## Fixture conflicts

**None.** The spec and the fixture agree on both built
tools' names, descriptions, required fields (`ticker`
for `ground_claim`; `id` for `verify_pack`) and property
shapes. The spec's note that the capture carries no
`title` keys matches the fixture exactly, and the
fixture's `_source` note records the same labelled
subset (2 kept, `screen_entity` and
`build_evidence_pack` omitted) with the same reasons.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned:
  `https://evidence.dropwatchhq.com/mcp`.
- Env override `COMPLIANCEEVIDENCE_MCP_URL` — labelled
  reconstruction.
- The registry record declares NO headers on the remote:
  no auth header is ever sent. Tests assert that no
  `Authorization` header is sent under any environment —
  including with key-like env vars
  (`COMPLIANCEEVIDENCE_API_KEY`, `API_KEY`,
  `AUTHORIZATION`) set — and that no `X-API-Key` header
  is ever sent; upstream requests carry only
  `Content-Type`, `Accept` and, when issued,
  `Mcp-Session-Id`.
- Error rendering `claimproof error:` text for upstream
  failures — labelled reconstruction.
- Upstream results pass through verbatim, including
  `isError` and extra fields. tools/call is forwarded as
  streamable-HTTP JSON-RPC: initialize (protocolVersion
  `2025-03-26`) → notifications/initialized → tools/call,
  honouring `Mcp-Session-Id` when issued; plain-JSON and
  SSE replies are both parsed. The forwarding core takes
  an injectable fetch; tests stub the upstream — no live
  network calls in tests.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with
  no `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3).
- Parses as JSON; `name` == `@reneturcios/claimproof`.
- Package entries: **128**.
- sha256: `25203f3c1815761cf52a1e1c8a602a73318a1e94d1c83058aa08c36bc6487fa0`
  (re-verified unchanged after the subsequent offline
  `npm ci`).

## server.json

- Description exactly: "Ground public-company claims
  against gov facts and verify signed compliance-evidence
  packs." — **91 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI
  entry (`ghcr.io/cloudcorprecords/claimproof:0.1.0`)
  carries **no** `version` field. Both entries carry
  packageArguments `[{type: positional, value: mcp}]` and
  transport stdio. Dockerfile (node:20-alpine, both
  stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/claimproof"`.
  Container workflow `IMAGE` is
  `ghcr.io/cloudcorprecords/claimproof` and its smoke test
  asserts the server name `claimproof`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli,
   forward, index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **57 tests, 57 pass, 0 fail**
   (registration-integrity suite deep-equals the served
   tools/list against the fixture's 2 tools, including the
   no-title assertions, plus the `screen_entity` and
   `build_evidence_pack` absence assertions at registry,
   MCP and CLI level, session handling/reuse, env
   override, the no-Authorization-header assertions under
   any environment, verbatim argument forwarding and
   result pass-through including upstream isError, SSE
   parsing, error rendering as `claimproof error:`,
   MCP-layer validation, and CLI behaviour for each
   subcommand).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/complianceevidence-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/claimproof/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/claimproof/parity-out` → **verdict PASS, parity 100.0**
   (2/2 identical, 0 missing, 0 extra, no schema diffs).
   Output in `parity-out/`.
