# mcp-registry-publisher

The single place that holds the faf.one DNS key and publishes `one.faf/*` servers to the MCP Registry.

## How a publish runs

1. A server repo publishes a GitHub release. Its own workflow runs the server's checks (version, packages live, mcpb asset).
2. It hands off here with `gh workflow run publish.yml -f repo=<owner/name> -f tag=<vX.Y.Z>`, using the `REGISTRY_PUBLISHER_DISPATCH_TOKEN` secret (fine-grained: this repo only, Actions: read and write).
3. `check` (no key) refuses anything not on the allowlist, not a published release, not named `one.faf/<repo>`, or whose `server.json` version is not the tag.
4. `publish` runs in the `registry` environment (main only, reviewer approval), logs in by DNS and publishes. If that version is already in the registry it logs in only, which proves the key, and publishes nothing.

## Secrets

| Secret | Where | What |
|---|---|---|
| `FAF_ONE_MCP_PRIVATE_KEY` | environment `registry` | 64-hex ed25519 seed; public half is the `v=MCPv1` TXT record at faf.one |
| `READ_TOKEN` | repo | fine-grained, Contents: read on the server repos (needed for private ones) |

Adding a server: add it to the `repo` options in `publish.yml`, give it the dispatch token, and give `READ_TOKEN` access if it is private.
