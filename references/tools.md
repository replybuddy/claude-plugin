# Tools of the ReplyBuddy MCP server

Server: `https://mcp.replybuddy.io/mcp` (all tools) or `https://mcp.replybuddy.io/mcp/readonly` (read tools only). Sign-in is OAuth 2.1; the consent screen picks the workspace and the access level (`read`, or `read` + `write`). Your MCP client shows the current tool list with descriptions and schemas.

## Two layers

1. **Typed tools** cover daily work and agent setup: the workspace profile, inbox, contacts, deals and tasks, deal funnel configuration, knowledge, QA, skills, goals, broadcasts, the sandbox and agent tests.
2. **Generic tools** reach every other operation of the REST API: find it with `search_api`, read its schema with `describe_api`, then call `api_read` (GET), `api_create` (POST), `api_update` (PUT or PATCH) or `api_delete` (DELETE). Paths are written without `/api/v1`, for example `/deal-fields`. The full reference is `https://mcp.replybuddy.io/openapi.json`.

`search_api` tells you when an operation already has a typed tool (`"tool": "create_goal"`); use that tool then. The generic write tools refuse operations that have a typed destructive tool (error `mcp.use_dedicated_tool`, the hint names the tool).

## How Claude asks before acting

Every tool is marked as read, write or destructive:

- **read** tools run without asking;
- **write** tools change data in a way you can undo inside ReplyBuddy (create, update, assign, move, toggle, restore); you can allow them once or always;
- **destructive** tools are irreversible or reach people and systems outside ReplyBuddy; Claude asks before each call. These are every `api_delete` call, tools that replace a whole set (`replace_*`, `update_deal_funnel_config`, `reset_deal_funnel_stages`, `sandbox_reset`), tools that write to clients (`send_message`, `resend_message`, broadcasts) and tools that send data to your systems (`create_webhook_endpoint`, `test_custom_api_tool`).

## Frequent generic calls

| Task | Call |
|---|---|
| Deal fields | `api_read /deal-fields`, `api_create /deal-fields`, `api_update /deal-fields/{id}`, `api_update /deal-fields/reorder`, `api_delete /deal-fields/{id}` |
| Deal stages | `api_read /deal-stages?funnel_id=`, `api_create /deal-stages`, `api_update /deal-stages/{id}`, `api_update /deal-stages/reorder`, `api_delete /deal-stages/{id}` |
| Deal funnels | `api_create /deal-funnels`, `api_update /deal-funnels/{id}` (rename), `api_create /deal-funnels/{id}/duplicate`, `api_delete /deal-funnels/{id}` |
| Task types | `api_read /task-types`, `api_create /task-types`, `api_update /task-types/{id}`, `api_delete /task-types/{id}` |
| Knowledge | `api_create /knowledge/{id}/toggle`, `api_update /knowledge/reorder`, `api_create /knowledge/{id}/restore`, `api_delete /knowledge/{id}` (archives) |
| QA | `api_create /qa/{id}/restore` |
| Skills | `api_delete /skills/{id}` |
| Goals | `api_read /funnel-stages/graph`, `api_update /funnel-stages/reorder`, `api_read /funnel-stages/{id}/followup-status`, `api_delete /funnel-stages/{id}` |
| Conversation goal | `api_update /conversations/{id}/funnel-stage` |
| Agent tests | `api_update /agent-tests/{id}`, `api_create /agent-tests/{id}/run`, `api_delete /agent-tests/{id}` |
| Sandbox | `api_read /agents/{id}/sandbox/state`, `api_create /agents/{id}/sandbox/regenerate`, `api_create /agents/{id}/sandbox/messages/{msg_id}/save-as-test` |
| Broadcast drafts | `api_read /agents/{id}/agent-broadcasts`, `api_create /agents/{id}/agent-broadcasts`, `api_update /agents/{id}/agent-broadcasts/{bid}`, then the typed `schedule_broadcast` or `send_broadcast` |
| Webhooks | see [webhooks.md](webhooks.md) |
| Session | `api_read /mcp/session` (workspace id and name, scopes, token expiry) |

Under `/agents/{id}` the connected workspace is used whatever id you pass.
