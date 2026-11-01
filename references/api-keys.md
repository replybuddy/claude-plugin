# API keys: your backend calls ReplyBuddy

An API key gives a server, a CI job or a headless agent access to one workspace without a browser sign-in.

## Creating a key

Only the workspace owner or an admin creates keys, in the ReplyBuddy dashboard (**Connections → API & MCP**). Claude cannot create, list or revoke keys: these routes are closed to every MCP token and key, so a token can never mint another token.

When creating a key you choose:

- a name, to tell keys apart;
- scopes: `read`, or `read` and `write`;
- an optional expiry; without one the key works until it is revoked.

The key (`rbmcp_key_…`) is shown once. Store it in your secret manager or an environment variable, never in a repository.

**A key acts as the user who created it, with that user's current role.** If that person leaves the workspace, the key stops working (403) and works again only if they return. Create integration keys from an account that will stay, and rotate by creating a new key, switching your backend to it, then revoking the old one.

## Calling the REST API

```bash
curl -H "Authorization: Bearer $REPLYBUDDY_API_KEY" \
  "https://mcp.replybuddy.io/api/v1/deals?limit=20"
```

- Base URL: `https://mcp.replybuddy.io/api/v1`. The operations and their schemas are in `https://mcp.replybuddy.io/openapi.json`; from Claude, `search_api` and `describe_api` show the same operations.
- The `agent_id` query parameter is filled in from the key. Passing another workspace's id is a 403.
- Paths of the form `/agents/{id}/…` need the workspace id; read it once from `GET /api/v1/mcp/session`.
- Errors have the shape described in [errors-and-limits.md](errors-and-limits.md); the rate limit is 200 requests per 10 seconds per client IP.
- A `read` key gets 403 `auth.insufficient_scope` on any write.

## Using a key for MCP without a browser

For CI or a headless agent, connect to the MCP server with the key as a static header instead of OAuth:

```bash
claude mcp add --transport http replybuddy https://mcp.replybuddy.io/mcp \
  --header "Authorization: Bearer $REPLYBUDDY_API_KEY"
```

Revoking a key stops data calls at once; the tool list may still be served for up to 60 seconds.
