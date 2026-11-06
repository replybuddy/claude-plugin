# Tools of the ReplyBuddy MCP server

Server: `https://mcp.replybuddy.io/mcp` (all tools) or `https://mcp.replybuddy.io/mcp/readonly` (read tools only). Sign-in is OAuth 2.1; the consent screen picks the workspaces and the access level (`read`, or `read` + `write`). Your MCP client shows the current tool list with descriptions and schemas.

## One tool per operation

Every operation has its own tool, named verb_object: `list_`, `get_`, `create_`, `update_`, `delete_` or `archive_`, and actions such as `move_deal`, `send_broadcast` or `rotate_webhook_secret`. There are about 200. Clients that load tools on demand (Claude Code, Codex) find them by name or task; the entry points (workspace, snapshot, conversations, deals, knowledge, goals, skills, sandbox) are always loaded. The REST reference for your own code is `https://mcp.replybuddy.io/openapi.json`.

## How the client asks before acting

Every tool is marked as read, write or destructive:

- **read** tools run without asking;
- **write** tools only add something new (`create_*`, `connect_skill`, runs and drafts); you can allow them once or always;
- **destructive** tools change or remove what exists, or reach people and systems outside ReplyBuddy, so the client asks before each call unless you allowed the tool. These are every update, move, toggle, assignment and archive, every delete, tools that replace a whole set (`replace_*`, `update_deal_funnel_config`, `save_all_deal_funnels`, `reset_deal_funnel_stages`, `sandbox_reset`), tools that write to clients (`send_message`, `edit_message`, broadcasts, marketing rules that answer on their own) and tools that send data to your systems (webhooks, `test_custom_api_tool`). A change you can undo is still destructive: that is how the plugin directories require it.

The markers describe the tools. Whether to ask is up to your client, and you can turn the questions off.

### Running without confirmations

- **Claude Code:** add `"mcp__plugin_replybuddy_replybuddy__*"` to `permissions.allow` in your settings. For a server you added yourself, write `mcp__<server name>__*`.
- **Claude.ai and Claude Desktop:** set the tool permissions in the connector settings.

Claude then sends messages and broadcasts to real clients and deletes data without asking. Use a read-only connection where you do not want that.

## Frequent tools beyond the basics

| Task | Tools |
|---|---|
| Deal fields | `list_deal_fields`, `create_deal_field`, `update_deal_field`, `reorder_deal_fields`, `delete_deal_field` |
| Deal stages | `list_deal_stages` (`funnel_id`), `create_deal_stage`, `update_deal_stage`, `reorder_deal_stages`, `delete_deal_stage` |
| Deal funnels | `create_deal_funnel`, `rename_deal_funnel`, `duplicate_deal_funnel`, `delete_deal_funnel` |
| Task types | `list_task_types`, `create_task_type`, `update_task_type`, `delete_task_type` |
| Knowledge | `toggle_knowledge`, `reorder_knowledge`, `archive_knowledge`, `restore_knowledge` |
| QA | `archive_qa`, `restore_qa` |
| Skills | `delete_skill` |
| Goals | `get_goal_graph`, `reorder_goals`, `get_goal_reminder_status`, `delete_goal` |
| Conversation goal | `set_conversation_goal` |
| Agent tests | `update_agent_test`, `run_agent_test`, `delete_agent_test` |
| Sandbox | `get_sandbox_state`, `regenerate_sandbox_reply`, `save_sandbox_reply_as_test` |
| Broadcasts | `list_broadcasts`, `create_broadcast`, `update_broadcast`, `count_broadcast_audience`, then `schedule_broadcast` or `send_broadcast` |
| Webhooks | see [webhooks.md](webhooks.md) |
| Analytics | `get_deal_analytics`, `get_user_analytics` (a period and a timezone) |

Under `/agents/{id}` the workspace of the call is used whatever id you pass.

## Connections with several workspaces

A connection may cover several workspaces. It then has `list_workspaces` (id, name and your role in each), and every other tool requires `agent_id`, set to one of those ids. Any other id gets `auth.access_denied`. A connection with one workspace has neither `list_workspaces` nor `agent_id`.
