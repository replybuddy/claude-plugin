# CRM: deals, tasks and funnel settings

A **deal** is a sale on a kanban board of a deal funnel. A conversation and its primary deal stand on the same stage: the AI or `set_conversation_stage` moves both, and `move_deal` moves the conversation with its deal, in its funnel or into another one. What the AI does on a stage is the stage's AI settings, see [workspace-model.md](workspace-model.md).

## Deals

| Task | Tools |
|---|---|
| Find and read | `list_deals` (filters), `get_deal`, `list_deal_events` (moves, field changes, links) |
| Change | `create_deal`, `update_deal`, `move_deal`, `delete_deal` |
| Contacts of a deal | `link_deal_contact`, `unlink_deal_contact`, `set_primary_deal_contact` |
| Conversation of a deal | `link_deal_conversation` (replaces the current one), `unlink_deal_conversation` |

## Tasks

Manager tasks have assignees and a due date. When the AI calls a person, the conversation gets a «needs human» mark and a reply task. The task and the mark go away when a person answers in the dashboard or in the messenger app, completes the task, sets the work state to `done` (`update_conversation_work_state`) or calls `dismiss_conversation_needs_human`; a reply sent through MCP or an API key keeps them.

- `list_tasks` lists tasks with filters, reply tasks and tasks the AI created included; `get_task_summary` counts open tasks by due group (overdue, today and so on). Without `funnel_id` both cover only the main funnel; tasks of another funnel need its `funnel_id`.
- `create_task` needs `assignee` or `assignees` and either `due_preset` or `due_at`. A calendar `due_at` (`YYYY-MM-DD`) and `due_time` (`HH:MM`) are read in the deal settings timezone; `timezone` may be omitted. On `update_task`, `assignees` replaces the whole list.
- `list_task_history` lists the changes of one task; `delete_task` deletes it.

## Funnel settings

Each deal funnel has its stages in board order, close reasons, card display and currency (`get_deal_funnel_config`, `update_deal_funnel_config`). Deal fields and task types are defined once for the workspace (`list_deal_fields`, `list_task_types`); a funnel's config chooses which fields it uses and shows on cards. `update_deal_stage` changes a stage's name, color and automations; its AI settings change with `update_stage_ai`. `save_all_deal_funnels` replaces the whole setup at once: the config of every funnel, deal fields and task types.

| Task | Tools |
|---|---|
| Funnels | `list_deal_funnels`, `create_deal_funnel`, `rename_deal_funnel`, `duplicate_deal_funnel`, `reorder_deal_funnels`, `delete_deal_funnel`, `reset_deal_funnel_stages` |
| Stages | `list_deal_stages`, `create_deal_stage`, `update_deal_stage`, `reorder_deal_stages`, `delete_deal_stage` |
| Deal fields | `list_deal_fields`, `create_deal_field`, `update_deal_field`, `reorder_deal_fields`, `delete_deal_field` |
| Task types | `list_task_types`, `create_task_type`, `update_task_type`, `delete_task_type` |
| Workspace deal settings | `get_deal_settings`, `update_deal_settings` |
