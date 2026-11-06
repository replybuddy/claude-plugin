---
name: configure-crm
description: Use when someone wants to shape the deal pipeline in ReplyBuddy's built-in CRM — "add a stage 'Invoice sent' after 'Qualified'", "we need a Budget field on deals", "create a second funnel for partners", "add a task type 'Demo'", "when a deal reaches 'Paid' create a task for the manager", "move deals automatically when the bot gets the phone number". Covers deal funnels, deal stages and their automations, deal fields, task types and close reasons.
---

# Configure the ReplyBuddy CRM

Deal funnels track sales deals on a kanban board. They are separate from the AI agent's goals (the API calls those funnel stages): a request about how the bot leads a dialog belongs to the `setup-sales-agent` skill. See [workspace-model.md](../../references/workspace-model.md).

## 1. Read the current setup

1. `list_deal_funnels` — every funnel; `is_main` marks the main one.
2. `get_deal_funnel_config` with the funnel `id` — its stages (`open`, then one `won` and one `lost`), `deal_fields` and `task_types` (both shared by the whole workspace), which fields the funnel shows (`fields`), `settings` (close reasons, currency, card display) and the two revisions `funnel.config_revision` and `workspace_revision`.
3. `list_goals` when the request links stages to the agent's goals.

## 2. Pick the way to change it

**Small changes: one record at a time, each with its own tool.** The tool's input schema shows the body:

| Change | Call |
|---|---|
| Add, rename or recolor a stage | `create_deal_stage` (`funnel_id` in `query`, defaults to the main funnel), `update_deal_stage`. A new stage goes after the last open stage; reorder to put it elsewhere |
| Reorder stages | `reorder_deal_stages`, body `ordered_ids`, `funnel_id` in `query`. `ordered_ids` must hold exactly the funnel's current open stages (no `won` / `lost`); read them with `list_deal_stages` right before the call. A 422 `deal.invalid_config` means the list differs from the stored one: read again and retry |
| Delete a stage | `delete_deal_stage` (its schema names where the stage's deals go) |
| Deal fields | `create_deal_field` (`label`, `type`: `text`, `number`, `date`, `phone`, `select` with `options`; numbers may be `number_mode: "currency"` with `currency_code`), `update_deal_field`, `reorder_deal_fields`, `delete_deal_field`. A new field is turned on in every funnel |
| Task types | `create_task_type` (`name`, `color`, `icon`), `update_task_type`, `delete_task_type` |
| New, renamed or copied funnel | `create_deal_funnel`, `rename_deal_funnel`, `duplicate_deal_funnel` |

**Restructuring a funnel: `update_deal_funnel_config`.** It saves the stages, field visibility and settings of one funnel atomically. It is destructive: it replaces the whole stage list, so build the body from `get_deal_funnel_config`, change what was asked, keep every other stage with its `id`, and send:

- `id`, `expected_revision` = `funnel.config_revision`;
- `stages` — the complete list; a stage without `id` is created, an existing stage left out is removed;
- `stage_transfers` — for each removed stage that has deals (`deals_count > 0`), where its deals go;
- `fields` and `settings` as read, with your changes;
- `deal_fields` and `task_types` only together with `expected_workspace_revision` = `workspace_revision`; then they replace the workspace lists, so send them complete.

Show the user the stage list before and after, and which deals move, before calling it. A 409 revision conflict means someone else changed the funnel: read it again and redo the change.

`reset_deal_funnel_stages` restores the default stages of the main funnel and moves its open deals onto them. Use it only when the user asks for a reset.

## 3. Automations on a stage

A stage may act when a deal enters it (`automation` in the stage body; `get_deal_funnel_config` returns it with each stage). Write `automation`, not the legacy `auto_task` field, and ignore `auto_task` in responses: it is a compatibility mirror and may show other values. Example: a call task for the deal owner due tomorrow is `{"automation": {"task": {"type_id": "<task type id>", "assignee": "deal", "due_preset": "tomorrow", "title": "Call the client"}}}`.

- `task` — create a task (`type_id` from `task_types`, `assignee`: `deal`, `ai` or a user id from `list_members`, `due_preset` or `due_days`);
- `reassign` — reassign the deal and/or the conversation;
- `message` — send the client a message from `template`. This writes to a real client on every deal that enters the stage: confirm the text with the user;
- `workspace_notification` — notify the team;
- `force_conversation_stage_id` — switch the dialog to a goal of the agent.

Links between deal stages and goals: `trigger_conv_stage_id` moves a deal into this stage when its dialog reaches that goal (only forward, never out of `won` or `lost`); `on_enter_conv_stage_id` sets the dialog's goal when a deal enters the stage. Use goal ids from `list_goals`. Moving a deal to `lost` requires an enabled close reason of that funnel.

## 4. Check

`get_deal_funnel_config` again: the stages, order, fields and automations must match the request. Tell the user what changed and where deals moved.

## Pitfalls

- Deal fields and task types belong to the workspace, not to one funnel; a funnel only decides whether a field is shown (`fields`).
- Every funnel has exactly one `won` and one `lost` stage at the end; new stages are `open`.
- Deleting a stage or funnel with deals needs a destination; use `update_deal_funnel_config` with `stage_transfers`, or `delete_deal_funnel`, whose schema names where its deals go.
- Errors carry `details.field`; see [errors-and-limits.md](../../references/errors-and-limits.md).
