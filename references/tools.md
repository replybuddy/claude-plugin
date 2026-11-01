# Tools of the ReplyBuddy MCP server

Server: `https://mcp.replybuddy.io/mcp` (all tools) or `https://mcp.replybuddy.io/mcp/readonly` (read tools only). Sign-in is OAuth 2.1; the consent screen picks the workspace and the access level (`read`, or `read` + `write`).

## Two layers

1. **Typed tools** cover daily work and agent setup (about 65 tools, listed below).
2. **Generic tools** reach every other operation of the REST API: find it with `search_api`, read its schema with `describe_api`, then call `api_read` (GET), `api_create` (POST), `api_update` (PUT or PATCH) or `api_delete` (DELETE). Paths are written without `/api/v1`, for example `/deal-fields`. The full reference is `https://mcp.replybuddy.io/openapi.json`.

`search_api` tells you when an operation already has a typed tool (`"tool": "create_goal"`); use that tool then. The generic write tools refuse operations that have a typed destructive tool (error `mcp.use_dedicated_tool`, the hint names the tool).

## How Claude asks before acting

Every tool is marked as read, write or destructive:

- **read** tools run without asking;
- **write** tools change data in a way you can undo inside ReplyBuddy; you can allow them once or always;
- **destructive** tools are irreversible or reach people outside ReplyBuddy; Claude asks before each call.

Destructive typed tools: `send_message`, `resend_message`, `schedule_broadcast`, `send_broadcast`, `cancel_broadcast`, `replace_conversation_tags`, `replace_contact_tags`, `replace_goal_graph`, `update_deal_funnel_config`, `reset_deal_funnel_stages`, `sandbox_reset`, `disconnect_account`, `test_custom_api_tool`, `create_webhook_endpoint`, and every `api_delete` call.

## Typed tools by area

| Area | Tools |
|---|---|
| Workspace | `get_workspace`, `update_workspace`, `get_workspace_snapshot`, `list_members`, `list_accounts`, `disconnect_account`, `create_webhook_endpoint` |
| Inbox | `list_conversations`, `get_conversation`, `list_messages`, `search_messages`, `send_message`, `resend_message`, `assign_conversation_ai`, `assign_conversation_operator`, `assign_conversation_user`, `update_conversation_work_state`, `replace_conversation_tags` |
| Contacts | `list_contacts`, `get_contact`, `create_contact`, `update_contact`, `replace_contact_tags`, `list_contact_notes`, `create_contact_note`, `list_tags`, `create_tag` |
| CRM | `list_deals`, `get_deal`, `create_deal`, `update_deal`, `move_deal`, `list_tasks`, `create_task`, `update_task` |
| Deal funnels | `list_deal_funnels`, `get_deal_funnel_config`, `update_deal_funnel_config`, `reset_deal_funnel_stages` |
| Agent | `list_knowledge`, `create_knowledge`, `update_knowledge`, `list_qa`, `create_qa`, `update_qa`, `list_skills`, `connect_skill`, `update_skill`, `toggle_skill`, `test_custom_api_tool`, `list_goals`, `create_goal`, `update_goal`, `toggle_goal`, `replace_goal_graph`, `list_history` |
| Broadcasts | `schedule_broadcast`, `send_broadcast`, `cancel_broadcast` (drafts are created with `api_create` on `/agents/{id}/agent-broadcasts`) |
| Testing | `sandbox_send_message`, `sandbox_list_messages`, `sandbox_reset`, `list_agent_tests`, `create_agent_test`, `run_agent_tests`, `get_agent_test_job` |

## Frequent generic calls

| Task | Call |
|---|---|
| Deal fields | `api_read /deal-fields`, `api_create /deal-fields`, `api_update /deal-fields/{id}`, `api_update /deal-fields/reorder`, `api_delete /deal-fields/{id}` |
| Deal stages | `api_read /deal-stages?funnel_id=`, `api_create /deal-stages`, `api_update /deal-stages/{id}`, `api_update /deal-stages/reorder`, `api_delete /deal-stages/{id}` |
| Deal funnels | `api_create /deal-funnels`, `api_update /deal-funnels/{id}` (rename), `api_create /deal-funnels/{id}/duplicate`, `api_delete /deal-funnels/{id}` |
| Task types | `api_read /task-types`, `api_create /task-types`, `api_update /task-types/{id}`, `api_delete /task-types/{id}` |
| Knowledge | `api_create /knowledge/{id}/toggle`, `api_update /knowledge/reorder`, `api_create /knowledge/{id}/restore`, `api_delete /knowledge/{id}` (archives) |
| QA | `api_create /qa/{id}/restore` |
| Goals | `api_read /funnel-stages/graph`, `api_update /funnel-stages/reorder`, `api_read /funnel-stages/{id}/followup-status`, `api_delete /funnel-stages/{id}` |
| Conversation goal | `api_update /conversations/{id}/funnel-stage` |
| Agent tests | `api_update /agent-tests/{id}`, `api_create /agent-tests/{id}/run`, `api_delete /agent-tests/{id}` |
| Sandbox | `api_read /agents/{id}/sandbox/state`, `api_create /agents/{id}/sandbox/regenerate`, `api_create /agents/{id}/sandbox/messages/{msg_id}/save-as-test` |
| Webhooks | see [webhooks.md](webhooks.md) |
| Session | `api_read /mcp/session` (workspace id and name, scopes, token expiry) |

Under `/agents/{id}` the connected workspace is used whatever id you pass.
