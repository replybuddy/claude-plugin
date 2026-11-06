---
name: diagnose-agent-reply
description: Use when a ReplyBuddy AI agent did something wrong or unexpected in a real dialog — "why did the bot answer that", "the agent didn't reply to this client", "it gave a wrong price", "it keeps asking the same question", "it didn't hand over to a manager", "reminders are not sent", "a message failed to deliver", "our webhook stopped receiving events". Finds the cause in the conversation, the agent's reasoning, the configuration and its change history, and proposes a fix.
---

# Diagnose an agent reply

Work from evidence: the messages, the agent's own reasoning, the configuration it ran with. Do not change anything until the user agrees with the diagnosis.

## 1. Find the conversation

- The user gave a name or username: `list_conversations` with `q`. Its items already carry `status`, `dialog_reply_mode`, `assigned_user_id`, `funnel_state` and the blocking flags.
- The user gave a link or id: `get_conversation`.
- The user remembers words from the dialog: `list_conversations` filtered by date, then `search_messages` inside the candidates.

Resolve `assigned_user_id` with `list_members` to name the person who holds the dialog.

## 2. Read the dialog

`list_messages` for the conversation (use `limit` and `offset`; `after_id` reads forward from a message). For every message note `direction` (`in` from the client, `out` from the agent or an operator), `is_ai_generated`, `status` and `created_at`. For AI messages read `thinking`: the agent's reasoning and a checklist of what it did (knowledge loaded, tags set, data fields collected, tools called). It is the most direct answer to "why".

## 3. Check the usual causes

**The agent did not reply**

- conversation `status: "manual"` — an operator took the dialog; the AI is silent until `assign_conversation_ai`;
- conversation `dialog_reply_mode` `manual` or `copilot`, or the workspace's `dialog_reply_mode` / `auto_reply_enabled` (`get_workspace`);
- `delivery_blocked`, `account_disabled` on the conversation, or the channel in `list_accounts` is not `active`;
- the reply failed to send: an `out` message with `status: "failed"` (or stuck in `sending`). `resend_message` sends it again (it reaches the client: ask first);
- the client's message never arrived: the last message is `out` and nothing `in` exists at the time the user says the client wrote. Then the problem is in the channel, not the agent: check the channel in the dashboard.

Before handing a dialog back to the AI, glance at the snapshot's `identity` and goals: if they do not fit this client (another business, another language), the agent's next replies will not fit either; say so.

**The agent answered wrongly**

- `get_workspace_snapshot` — the configuration exactly as the runtime receives it. Look for the source of the wrong answer: identity, a behavior section, the current goal's `instruction`, a knowledge entry (`list_knowledge`), a QA pair (`list_qa`).
- `thinking` says which knowledge it loaded; a missing entry means the `usage_prompt` did not match the question.
- No source for a fact means the agent made it up: the fact belongs in knowledge or QA, and the identity should forbid guessing it.
- Was the configuration different at that time? `list_history` with `entity_type: "profile"` (or `knowledge` / `qa` with `entity_id`) shows versions with dates; compare them with the message time.

**Goals and reminders**

- `funnel_state` on the conversation is the current goal. Wrong goal: compare `when_to_enter` of the goals in the snapshot; the runtime picks goals by `when_to_enter`, not by graph edges.
- Asks the same question again: the field may be collected under another `code`, or the goal's `stop_condition` never matches.
- Reminders: the goal's `cadence` and `working_hours`, the conversation's `followups_disabled`, and `get_goal_reminder_status` (queued reminders and when a pending change applies).

**Handoff did not happen**

The `handoff_escalate` and `handoff_notify` sections in the snapshot's `behavior`. Vague conditions ("when needed") are the usual cause.

**A custom_api tool misbehaved**

`thinking` shows the call; `list_skills` shows the tool; reproduce with `test_custom_api_tool` and the same arguments (a real request: ask first).

**Webhooks stopped**

`list_webhook_endpoints` (`enabled`, `disabled_reason`), then `list_webhook_deliveries` with `status: "failed"`: `last_status` and `last_error` show what the receiver answered. See [webhooks.md](../../references/webhooks.md).

## 4. Report and fix

1. State the cause with the evidence: message ids, the line of `thinking`, the configuration field.
2. Propose the smallest fix and where it goes (which section, entry or goal). For the profile and goals, use the `setup-sales-agent` skill.
3. After the user agrees and the fix is applied, reproduce the client's message in the sandbox with the `test-agent` skill and, for a recurring problem, save a test case.

## Pitfalls

- `system_prompt` in the snapshot is legacy and unused.
- The snapshot shows the configuration now, not at the time of the dialog; use `list_history` for the past.
- An `auth.access_denied` error on an id that looks right usually means the record does not exist in this workspace.
- Conversations contain clients' personal data. Quote only what the diagnosis needs.
