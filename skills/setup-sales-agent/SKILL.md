---
name: setup-sales-agent
description: Use when someone wants their ReplyBuddy AI agent to sell, consult or qualify leads in a messenger and needs it set up or reworked — "set up our bot for the new course", "the agent should collect the phone number and hand over to a manager", "load our price list and FAQ", "make it sound less robotic", "add a goal for booking a call". Covers the agent's identity and behavior, knowledge, QA answers, goals with data to collect and reminders, and catalog skills, ending with a check in the sandbox.
---

# Set up a ReplyBuddy sales agent

The workspace model (profile, behavior sections, knowledge, QA, skills, goals, and how goals differ from CRM deal stages) is in [workspace-model.md](../../references/workspace-model.md). Read it first if the terms are new.

## 1. See what is there

1. `get_workspace_snapshot` — the configuration exactly as the AI runtime receives it. Note `auto_reply_enabled`, `dialog_reply_mode`, the enabled skills, the goals and `funnel_start_stage_id`.
2. `list_accounts` — which messengers are connected. Formatting rules depend on them (Zalo and some others show no Markdown).
3. `list_knowledge`, `list_qa` (and `list_qa` with `has_answer: false` for questions the agent saved but could not answer), `list_goals`.

Summarize to the user what the agent does today before proposing changes. If the workspace already has a working setup, change it in place rather than rebuilding it. Point out parts that contradict each other, such as an identity for one business and goals for another.

For one small change (a knowledge entry, a QA pair, one behavior section) the snapshot and the list of that kind are enough; skip the full survey.

## 2. Collect the facts from the user

Ask for what the agent must not invent: the product or service, prices and plans, conditions and guarantees, contacts, working hours, what counts as a qualified lead, which data to collect, and when a human must take over. Take facts from documents, a website or files the user points to. Never fill gaps with plausible numbers; leave a gap and say so.

## 3. Write the configuration

Do the steps the request needs, in this order:

1. **Identity** (`update_workspace` → `identity`): who the agent is, for whom it works, tone, language, hard rules. It replaces the whole text: show the user the new text, or the diff against the current one, before saving.
2. **Behavior** (`update_workspace` → `behavior`): send only the sections you change. `handoff_escalate` says when the agent stops and hands the dialog to a human; `handoff_notify` when it alerts a human but keeps replying; `formatting`, `message_split`, `reply_policy`, `followups` shape how it writes. Each section is at most 4000 characters.
3. **Knowledge** (`create_knowledge`): one topic per entry, with `usage_prompt` saying when to use it ("when the client asks about prices or plans"). Update existing entries with `update_knowledge` instead of adding duplicates.
4. **QA** (`create_qa`): exact answers for frequent questions, or an `instruction` for how to answer one.
5. **Goals** (`create_goal`, `update_goal`): `name`, `instruction` (what the agent does while pursuing it), `stop_condition` (when it is reached), `when_to_enter` (when the agent takes it up), `data_fields` to collect (`code`, `description`, `value_type`, `required`), and `cadence` for reminders when the client goes quiet. Omit `cadence` for a goal without reminders.
   - To make a goal the start goal, first `api_read /funnel-stages/graph`, then call `replace_goal_graph` with the same `transitions` and `layout` and the new `start_stage_id`. The tool replaces the whole graph, so sending only `start_stage_id` would delete the edges.
   - The runtime picks goals by `when_to_enter`, not by graph edges; write `when_to_enter` so that two goals do not compete for the same moment.
6. **Catalog skills** (`connect_skill`): `capture_question` to save questions the agent could not answer, `web_search`, `schedule_meeting` (needs a Calendly or Google Calendar token from the user). Every skill needs `when_to_use`. For calls to the business's own API use the `connect-backend` skill.
7. **Reply mode** last: `dialog_reply_mode` `auto` replies by itself, `copilot` drafts for a human, `manual` stays silent. Turning on `auto` makes the agent answer real clients on every connected channel: confirm it with the user.

Use `search_api` for anything not covered here (toggling, reordering or restoring knowledge, deleting a goal); [tools.md](../../references/tools.md) lists the frequent paths.

## 4. Check the result

1. `get_workspace_snapshot` again and compare with step 1: the runtime must see what you intended.
2. Run the `test-agent` skill: a short sandbox dialog that walks the main goal, plus test cases for the questions that matter most.
3. Report what changed, what the sandbox showed, and which facts are still missing.

## Pitfalls

- `system_prompt` in the snapshot is legacy and unused; do not write instructions there.
- Goals and CRM deal stages are different things. "Funnel" in a sales request usually means goals; a kanban board for deals is the `configure-crm` skill.
- `api_delete /knowledge/{id}` archives an entry; `api_create /knowledge/{id}/restore` brings it back. `api_create /knowledge/{id}/toggle` turns an entry off without archiving it.
- Long identities that repeat the knowledge base make replies slower and less consistent. Keep facts in knowledge and QA, and rules in identity and behavior.
- If an error comes back, read `code` and `details.field` and fix that argument ([errors-and-limits.md](../../references/errors-and-limits.md)).
