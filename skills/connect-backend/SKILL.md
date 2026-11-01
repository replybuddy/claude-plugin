---
name: connect-backend
description: Use when a developer wants ReplyBuddy and their own system to talk to each other — "the bot should check order status in our API", "send new leads and messages to our CRM", "notify our backend when a deal moves", "call ReplyBuddy from our server", "verify the webhook signature", "why are webhooks not arriving". Covers custom_api skills (the agent calls your API), signed webhooks (ReplyBuddy calls your backend) and API keys (your backend calls ReplyBuddy).
---

# Connect ReplyBuddy to your backend

Three directions, three mechanisms. Find out which ones the user needs before building anything:

| Direction | Mechanism | Details |
|---|---|---|
| The AI agent calls your API during a dialog | `custom_api` skill | [custom-api-skills.md](../../references/custom-api-skills.md) |
| ReplyBuddy notifies your backend about events | Webhook endpoint | [webhooks.md](../../references/webhooks.md) |
| Your backend reads or changes ReplyBuddy data | API key + REST API | [api-keys.md](../../references/api-keys.md) |

When the user's code is in the current project, read it first: existing routes, auth, the framework's way to read a raw request body. Match it.

## The agent calls your API (custom_api)

1. `list_skills` — reuse an existing `custom_api` skill for the same system instead of adding a second one. Tool names are unique across the workspace.
2. Agree with the user on each endpoint: method, URL, parameters and where each value comes from in the dialog, authentication. Ask for the token; never invent one.
3. `connect_skill` with `skill_type: "custom_api"`, a `when_to_use` that says in which client situations to call it, and `is_enabled: false` while testing.
4. `test_custom_api_tool` with the skill `id`, the tool object from the `connect_skill` response and sample `arguments`. It sends one real request: use a read-only endpoint or a staging backend for anything that changes data, and say so before calling it. Fix the URL, parameters or auth until the status and body are right. `502 skill.test_request_failed` means no answer came from the business API; check that the URL is public and reachable.
5. `toggle_skill` to enable it, then check with the `test-agent` skill that the agent calls the tool when it should. The `thinking` field of the sandbox reply shows the call. Sandbox calls of a `custom_api` tool are real requests, like `test_custom_api_tool`.
6. To change a tool later, `update_skill` with all tools of the skill, their names unchanged and secrets sent as `******` (or left out) to keep them. Remove a skill with `api_delete /skills/{id}`.

## ReplyBuddy notifies your backend (webhooks)

1. Agree on the events (`message.received`, `conversation.handoff_requested`, `deal.created`, `deal.updated`, `deal.moved`, `deal.deleted`) and a public `https` URL. The URL cannot be changed later.
2. If the receiver does not exist yet, write it in the user's stack: verify `X-ReplyBuddy-Signature` over the raw body, reject timestamps older than 5 minutes, deduplicate on the envelope `id`, answer `2xx` fast and process asynchronously. The exact algorithm and tested Python and Node.js versions are in [webhooks.md](../../references/webhooks.md).
3. `create_webhook_endpoint` with `url` and `events`. Claude asks before it because a new endpoint starts sending workspace data outside (client message texts with `message.received`): confirm the URL and the events with the user, and subscribe only to what was asked. The response has the signing `secret` once: tell the user to put it into their secret store or environment variable now; do not write it into tracked files.
4. `api_create /webhook-endpoints/{id}/test` sends a `webhook.test` event right away; check `status`, `last_status` and `last_error` in the returned delivery (`last_status` is absent when the receiver did not answer at all; `last_error` then says why).
5. Later: `api_read /webhook-endpoints/{id}/deliveries` for the log, `api_create /webhook-deliveries/{id}/redeliver` to send one again, `api_update /webhook-endpoints/{id}` with `enabled: true` to resume an endpoint disabled after repeated failures, `api_create /webhook-endpoints/{id}/rotate-secret` to rotate.

## Your backend calls ReplyBuddy (API keys)

1. A key is created only by the workspace owner or an admin in the dashboard (Connections → API & MCP); Claude cannot create one. Tell the user where and which scopes to choose, and that the key acts as its creator and stops working if that person leaves the workspace.
2. Write the client code with `Authorization: Bearer <key>` against `https://mcp.replybuddy.io/api/v1`, the key read from an environment variable.
3. Find operations with `search_api` and their exact schemas with `describe_api`; generate request and response types from `describe_api` output, not from guesses. The full spec is `https://mcp.replybuddy.io/openapi.json`.
4. To try a call before writing code, make the same call with `api_read` / `api_create` / `api_update` from Claude.

## Pitfalls

- `api_create` refuses `POST /webhook-endpoints` (`mcp.use_dedicated_tool`): use `create_webhook_endpoint`.
- `message.received` fires for incoming client messages only. To follow the agent's replies, read `list_messages` from your backend with an API key.
- Webhook delivery is at least once and may repeat; a receiver without deduplication creates duplicate leads.
- A redirect (3xx) from the webhook URL counts as a failure; point to the final URL.
- An endpoint is disabled after about 28 hours of failures; check `enabled` and `disabled_reason` with `api_read /webhook-endpoints` when events stop.
