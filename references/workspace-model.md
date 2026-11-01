# How a ReplyBuddy workspace is put together

A **workspace** is one AI agent together with its messenger channels, contacts, inbox and CRM. One MCP connection covers the workspaces ticked on the consent screen when you sign in. With one workspace, tools take no workspace or agent id: the server fills it in from the token. With several, call `list_workspaces` and pass the chosen id as `agent_id`. If the request does not make clear which workspace it is about, ask the user before changing anything.

## What the agent's reply is built from

| Part | What it is | Tools |
|---|---|---|
| Profile | `identity` (who the agent is, tone, language, rules), reply mode, reply quality, media and voice settings, restrictions | `get_workspace`, `update_workspace` |
| Behavior sections | Six short instructions, each up to 4000 characters: `formatting`, `message_split`, `reply_policy`, `handoff_notify` (when to notify a human but keep replying), `handoff_escalate` (when to stop and hand the dialog to a human), `followups` (how reminders should sound) | `update_workspace` with `behavior: {...}`; only the sections you send change |
| Knowledge | Documents the agent searches while answering. Each entry has `title`, `data` and `usage_prompt` (when to use it) | `list_knowledge`, `create_knowledge`, `update_knowledge`; toggle, reorder, restore and archive through `api_*` |
| QA pairs | Exact answers or instructions for frequent questions | `list_qa`, `create_qa`, `update_qa`; restore through `api_*` |
| Skills | Actions the agent can take: `capture_question` (save questions it could not answer), `web_search`, `schedule_meeting` (Calendly or Google Calendar), `custom_api` (calls your HTTP API; several per workspace) | `list_skills`, `connect_skill`, `update_skill`, `toggle_skill`, `test_custom_api_tool` |
| Goals | What the agent pursues in a dialog: `name`, `instruction`, `stop_condition`, `when_to_enter`, data fields to collect and a reminder cadence for when the client goes quiet | `list_goals`, `create_goal`, `update_goal`, `toggle_goal`, `replace_goal_graph` |

`get_workspace_snapshot` returns all of this exactly as the AI runtime receives it, with skill secrets masked. Use it before and after a change. Its `system_prompt` field is legacy and unused; `config_version_id` grows by one on every change.

## Goals are not deal stages

Two different things are both called "funnel" in the product:

- **Goals** (the API calls them funnel stages: `/funnel-stages`, `start_stage_id`, `funnel_state` on a conversation) steer the AI inside one dialog. The runtime takes up a goal by its `when_to_enter`; it does not walk the edges of the goal graph. The start goal is the one a new dialog begins with.
- **Deal stages** belong to CRM deal funnels (`/deal-funnels`, `/deal-stages`). They track sales deals on a kanban board. A deal stage may be linked to a goal (`trigger_conv_stage_id` moves a deal when the dialog reaches that goal; `on_enter_conv_stage_id` forces a goal when a deal enters the stage).

## Reply modes

- `auto_reply_enabled` on the workspace switches AI replies on or off as a whole.
- `dialog_reply_mode`: `auto` (the AI replies), `copilot` (the AI drafts, a human sends), `manual` (no AI replies). A conversation may override it with its own `dialog_reply_mode`.
- A conversation with `status: "manual"` is handled by an operator and the AI stays silent until `assign_conversation_ai`.
- `reply_quality`: `fast`, `standard`, `thorough` or `auto`; slower modes think longer before replying.

## What only the dashboard does

Connecting messenger channels (Telegram, Instagram, VK, Avito, MAX, Zalo), inviting members, creating API keys and deleting the workspace are done in the ReplyBuddy dashboard at https://replybuddy.io. No MCP token can reach these operations.
