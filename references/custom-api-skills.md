# custom_api skills: the AI agent calls your API

A `custom_api` skill gives the AI agent functions it may call while talking to a client: look up an order, check stock, create a booking. A workspace may have several `custom_api` skills; each holds one or more tools (endpoints).

## Shape

```json
{
  "skill_type": "custom_api",
  "when_to_use": "When the client asks where their order is and gives an order number.",
  "is_enabled": false,
  "config": {
    "name": "Order status",
    "tools": [{
      "name": "get_order_status",
      "description": "Returns status and delivery date of one order by its number.",
      "method": "GET",
      "url": "https://api.example.com/orders/{order_id}",
      "parameters": [
        {"name": "order_id", "location": "path", "dataType": "string", "required": true, "description": "Order number the client gave, digits only."}
      ],
      "authentication": "none"
    }]
  }
}
```

Rules the server checks:

- `when_to_use` is required: the agent decides from it whether to use the skill at all.
- Tool `name` and parameter `name` are snake_case. A tool name is unique across all `custom_api` skills of the workspace.
- Every `{placeholder}` in `url` needs a parameter with that name and `location: "path"`. Other locations: `query`, `body`, `header`.
- A `{placeholder}` goes in the path or query, never in the host. A URL with `user:password@` is refused; use `authentication` instead.
- `dataType`: `string` (default), `number`, `boolean`. `required` defaults to `true`.
- `authentication`: `none` (default), `bearer`, `api_key` (header in `apiKeyHeader`, default `X-Api-Key`), `basic` (`basicUsername`). The key, token or password itself is entered by the user in the dashboard, see Secrets. Constant headers go to `fixedHeaders: [{"name", "value"}]`.
- JSON keys inside a tool are camelCase (`apiKeyHeader`, `dataType`, `fixedHeaders`).

## Secrets

Secrets are entered only in the ReplyBuddy dashboard: the user opens the skill under Agent → Skills, chooses the authentication and types the key, token or password there. A request through this connection that carries a secret is refused with 422 `skill.secret_in_dashboard`, so it never passes through the chat. The same applies to the `schedule_meeting` calendar token. `fixedHeaders` and the URL are ordinary settings, returned as written: keep secrets out of them.

Responses mask secrets as `******`. On `update_skill`, leave them as `******` or leave them out to keep the stored value while the URL keeps its scheme and host. A stored secret never moves to another host: after changing the host, the call fails with 422 `skill.invalid_config` until the user enters the secret again in the dashboard. Send every tool of the skill on update with its `name` unchanged (and its `id` when `list_skills` shows one): a stored secret is matched to its tool by `id`, then by `name`. Delete a skill with `delete_skill`. ReplyBuddy stores the secrets and uses them only to call the tool's URL; they never come back through the API.

## Testing a tool

`test_custom_api_tool` sends one real request through a tool of a saved skill and returns the status, headers and body. Pass the skill `id`, the tool object as `connect_skill` or `list_skills` returned it (you may edit it before the test) and sample `arguments`. A `502 skill.test_request_failed` means the request did not get an answer from your API (DNS, TLS, timeout, or the address is not allowed); it is not a ReplyBuddy outage. Stored secrets are used on the server while the URL keeps the stored scheme and host. Private and internal addresses are refused; the limit is 20 calls per minute.

This is a real request to your API: test against a read-only endpoint or a staging backend when the call changes data.

## Writing good tools

- One tool per business action, named the way a person would describe it (`get_order_status`, not `orders_v2_get`).
- The `description` says when to call it and what comes back; parameter descriptions say where the value comes from ("the order number the client wrote").
- Return compact JSON with the fields the agent needs to answer. Large or nested responses slow the reply and confuse the agent.
- Answer errors with a clear message ("order not found") and a 4xx status, so the agent can tell the client.
