# How a ReplyBuddy workspace is put together

A **workspace** is one AI agent together with its messenger channels, contacts, inbox and CRM. One MCP connection covers the workspaces ticked on the consent screen when you sign in. With one workspace, tools take no workspace or agent id: the server fills it in from the token. With several, call `list_workspaces` and pass the chosen id as `agent_id`. If the request does not make clear which workspace it is about, ask the user before changing anything.

## What the agent's reply is built from

| Part | What it is | Tools |
|---|---|---|
| Profile | `identity` (who the agent is, tone, language, rules), reply mode, reply quality, media and voice settings, restrictions | `get_workspace`, `update_workspace` |
| Behavior sections | Short instructions, each up to 4000 characters: `formatting`, `message_split`, `reply_policy`, `handoff_notify` (when to notify a human but keep replying), `handoff_escalate` (when to stop and hand the dialog to a human), `followups` (how reminders sound and when to turn them off or on), `deals` (when to put a person on a stage) | `update_workspace` with `behavior: {...}`; only the sections you send change |
| Knowledge | Documents the agent searches while answering. Each entry has `title`, `data` and `usage_prompt` (when to use it) | `list_knowledge`, `create_knowledge`, `update_knowledge`; toggle, reorder, restore and archive through `api_*` |
| QA pairs | Exact answers or instructions for frequent questions | `list_qa`, `create_qa`, `update_qa`; restore through `api_*` |
| Skills | Actions the agent can take: `capture_question` (save questions it could not answer), `web_search`, `schedule_meeting` (Calendly or Google Calendar), `custom_api` (calls your HTTP API; several per workspace) | `list_skills`, `connect_skill`, `update_skill`, `toggle_skill`, `test_custom_api_tool` |
| Stage AI settings | What the agent does while a dialog stands on a deal stage: `instruction`, `stop_condition`, `when_to_enter`, data fields to collect and a reminder cadence for when the client goes quiet | `get_stage_catalog`, `update_stage_ai`, `create_deal_stage` with `ai` |

`get_workspace_snapshot` returns all of this exactly as the AI runtime receives it, with skill secrets masked. Use it before and after a change. Its `system_prompt` field is legacy and unused; `config_version_id` grows by one on every change.

## Stages, deals and dialogs

- **Deal stages** belong to CRM deal funnels (`/deal-funnels`, `/deal-stages`) and show deals on a kanban board. A dialog stands on the stage of its open (primary) deal; `funnel_state` on a conversation names it. The AI moves the dialog and its deal together, and `move_deal` or `set_conversation_stage` by a person does the same.
- The **incoming stage** («no deal» in the dashboard) holds dialogs without an open deal. It has AI settings like any other stage and is listed only by `get_stage_catalog`.
- The `deals` behavior section tells the agent when to open a deal. While it is filled the AI may enter any open stage; a stage's `when_to_enter` narrows its own entry, and a `won` or `lost` stage is entered only by its own `when_to_enter`.
- A stage is a processing status, not a request type: request types are tags plus QA instructions.
- When the agent calls a person, the dialog gets a task «reply in dialog», completed when someone answers or dismisses the mark.

## Reply modes

- `auto_reply_enabled` on the workspace switches AI replies on or off as a whole.
- `dialog_reply_mode`: `auto` (the AI replies), `copilot` (the AI drafts, a human sends), `manual` (no AI replies). A conversation may override it with its own `dialog_reply_mode`.
- A conversation with `status: "manual"` is handled by an operator and the AI stays silent until `assign_conversation_ai`.
- `reply_quality`: `fast`, `standard`, `thorough` or `auto`; slower modes think longer before replying.

## What only the dashboard does

Connecting messenger channels (Telegram, Instagram, Avito, MAX, Zalo), inviting members, creating API keys and deleting the workspace are done in the ReplyBuddy dashboard at https://replybuddy.io. No MCP token can reach these operations.
