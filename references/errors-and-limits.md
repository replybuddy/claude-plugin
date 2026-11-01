# Errors and limits

## Error shape

A failed call is an ordinary tool result with `isError: true` and this text:

```json
{"error": {"status": 400, "code": "validation.invalid_query", "message": "...", "details": {"field": "limit"}, "hint": "..."}}
```

`code` is stable and safe to branch on; `details.field` names the argument to fix; `hint` appears for a read-only token, a forbidden route, the rate limit, an oversized result and server errors. Read the error, fix the call, retry once. Do not retry the same arguments in a loop.

| Code | Meaning | What to do |
|---|---|---|
| `validation.field_required`, `validation.invalid_body`, `validation.invalid_query` | An argument is missing or wrong; `details.field` says which | Fix that argument; check `describe_api` for generic calls |
| `funnel.field_required`, `skill.invalid_config` | The goal or skill body is incomplete | Add the field from `details.field` |
| `deal.invalid_config` (422) | A deal stage, funnel or reorder body does not match the stored funnel (for reorder: not exactly the current open stages) | Read the funnel again and rebuild the body |
| `skill.test_request_failed` (502) | `test_custom_api_tool` got no answer from your API | Check that the URL is public, resolves and answers over TLS |
| `skill.already_connected` (409) | That catalog skill is already on the workspace | `list_skills`, then `update_skill` |
| `auth.insufficient_scope` (403) | The connection has the `read` scope only | Reconnect with write access, or use `/mcp` instead of `/mcp/readonly` |
| `auth.mcp_route_forbidden` (403) | The operation is dashboard-only (channels, members, API keys, workspace deletion) | Do it at https://replybuddy.io |
| `auth.access_denied` (403) | Not your workspace, your role lacks the permission, or the id does not exist | Check the id with a list tool; check the user's role |
| `mcp.use_dedicated_tool` | A generic write was used for an operation that has a typed destructive tool | Call the tool named in `hint` |
| `mcp.result_too_large` | The result is over 40 000 characters | Pass a smaller `limit`, or narrow the filter |
| `rate_limited` (429) | More than 200 requests in 10 seconds from your IP | Wait `Retry-After` seconds |

## Limits

- 200 requests per 10 seconds per client IP, on every MCP host path including sign-in.
- A tool result is at most 40 000 characters. List tools take `limit` (knowledge and QA 1–100, CRM lists 1–200); out-of-range values are rejected, not clamped.
- `test_custom_api_tool`: 20 calls per minute; private and internal addresses are refused.
- Access tokens live 1 hour and refresh tokens 30 days; Claude refreshes them for you. A revoked connection may still list tools for up to 60 seconds, while data calls fail at once.
- A workspace has at most 10 webhook endpoints.
