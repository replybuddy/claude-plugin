# Errors and limits

## Error shape

A failed call is an ordinary tool result with `isError: true` and this text:

```json
{"error": {"status": 400, "code": "validation.invalid_query", "message": "...", "details": {"field": "limit"}, "hint": "..."}}
```

`code` is stable and safe to branch on; `details.field` names the argument to fix; `hint` appears for a read-only token, a forbidden route, the rate limit, an oversized result and server errors. Read the error, fix the call, retry once. Do not retry the same arguments in a loop.

| Code | Meaning | What to do |
|---|---|---|
| `validation.field_required`, `validation.invalid_body`, `validation.invalid_query` | An argument is missing or wrong; `details.field` says which | Fix that argument; the tool's input schema lists the arguments |
| `skill.field_required`, `skill.invalid_config` | The skill body is incomplete | Add the field from `details.field` |
| `funnel.invalid_config`, `funnel.field_code_required`, `funnel.field_code_duplicate_in_payload` | The AI settings of a stage are invalid, or a data field has no code or repeats one | Fix the `update_stage_ai` body |
| `funnel.field_type_conflict` (409) | A data field code already has another value type on another stage; `details.field` names it | Reuse the type from `get_stage_field_schema`, or pick another code |
| `deal.invalid_config` (422) | A deal stage, funnel or reorder body does not match the stored funnel (for reorder: not exactly the current open stages), or `stage_id` is not a stage of this workspace; `details.field` says which | Read the funnel again and rebuild the body |
| `deal_task.invalid_config`, `deal_task.invalid_due_at`, `deal_task.invalid_due_time` (422) | A task argument is wrong, or `type_id` / `deal_id` is not an id of this workspace; `details.field` says which | Fix that argument; read ids with `list_task_types` or `list_deals` |
| `broadcast.preview_invalid` (422) | The preview needs an AI broadcast and a `conversation_id` of the workspace; `details.field` says which | Pass a conversation from the audience, or skip the preview for a static broadcast |
| `message.hide_unsupported`, `message.edit_incoming`, `message.edit_unsupported`, `message.delete_unsupported`, `message.already_deleted`, `platform.unsupported` (400) | The message cannot be hidden, edited or deleted on its platform | Do not retry; only incoming Instagram comments can be hidden |
| `message.conversation_mismatch` (400) | The message belongs to another conversation | Take the message id from `list_messages` of that conversation |
| `deal.amount_out_of_range` (422) | The deal `amount` is outside 0 to 999999999999.99 | Ask the user for the correct amount |
| `skill.test_request_failed` (502) | `test_custom_api_tool` got no answer from your API | Check that the URL is public, resolves and answers over TLS |
| `skill.secret_in_dashboard` (422) | A key, token or password was sent through this connection | Save the skill without it and ask the user to enter it in the dashboard under Agent → Skills |
| `skill.already_connected` (409) | That catalog skill is already on the workspace | `list_skills`, then `update_skill` |
| `auth.insufficient_scope` (403) | The connection has the `read` scope only | Reconnect with write access, or use `/mcp` instead of `/mcp/readonly` |
| `auth.mcp_route_forbidden` (403) | The operation is dashboard-only (channels, members, API keys, workspace deletion) | Do it at https://replybuddy.io |
| `auth.access_denied` (403) | Not your workspace, your role lacks the permission, or the id in the tool's path (`id`, `msgId`, ...) does not exist; an unknown id in the arguments body is a 422 naming the field | Check the id with a list tool; check the user's role |
| `mcp.result_too_large` (413) | A call without paging returned over 40 000 characters (a paged list instead returns the items that fit and a note with the arguments that continue it) | Narrow the filter |
| `skill.test_rate_limited` (429) | More than 20 `test_custom_api_tool` calls a minute | Wait a minute and retry |

## Limits

- 200 requests per 10 seconds per client IP on the MCP host, sign-in included. Past it the HTTP request itself gets 429 with `Retry-After`, not a tool result.
- A tool result is at most 40 000 characters. List tools take `limit`; the tool's schema gives its range (knowledge and QA 1–100; deals, contacts and tasks 1–200; most other lists up to 100). Stay inside it: some lists reject a value out of range, others fall back to their default.
- `test_custom_api_tool`: 20 calls per minute; private and internal addresses are refused.
- Access tokens live 1 hour and refresh tokens 30 days; the client refreshes them. A revoked connection may still list tools for up to 60 seconds, while data calls fail at once.
- A workspace has at most 10 webhook endpoints.
