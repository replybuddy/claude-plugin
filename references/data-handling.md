# What data goes where

All data moves through the ReplyBuddy MCP server, and only when the AI client calls one of its tools. The ReplyBuddy plugin for Claude contains only instructions (Markdown) and one server address (JSON): it runs no code, collects nothing and sends nothing on its own.

## Connections

| From → to | What | When |
|---|---|---|
| AI client → `https://mcp.replybuddy.io` | OAuth sign-in, then MCP requests over HTTPS with a bearer token | Every tool call |
| ReplyBuddy → AI client | Tool results: workspace settings, knowledge, conversations and messages, contacts (names, usernames, phone numbers and other fields your clients shared), deals, tasks | When the client reads them for your request |
| AI client → ReplyBuddy | The changes you ask for: settings, knowledge, QA, stage settings, skills, CRM records, messages to clients, broadcasts | Write and destructive tools; clients ask before destructive ones unless you allowed them |
| ReplyBuddy → your API | Requests from `custom_api` skills while the agent talks to clients, and one request per `test_custom_api_tool` call | Only to URLs you configured |
| ReplyBuddy → your webhook URL | Signed event POSTs, including the text of incoming client messages | Only for endpoints you created |
| ReplyBuddy → clients' messengers | Messages and broadcasts | For `send_message`, `resend_message`, `edit_message`, `delete_message`, `schedule_broadcast`, `send_broadcast`, `resume_broadcast`, `set_comment_hidden`, the agent's own replies, and marketing rules and lead forms you create, which then message clients on their own |

What ReplyBuddy returns becomes part of your conversation with the AI client and is handled under your agreement with its provider (for Claude, Anthropic). Ask for aggregates or a few records when you do not need full client data.

## Access

- The connection covers only the workspaces you tick on the consent screen, and acts with your role in each. The access level applies to all of them, and revoking the connection removes access to all of them. With several workspaces, the client can see data from all of them in one conversation, including client messages: tick only the ones you need.
- On the consent screen you grant `read`, or `read` and `write`. Connect to `https://mcp.replybuddy.io/mcp/readonly` for a connection that can never change anything.
- Access tokens expire after 1 hour, refresh tokens after 30 days. You can disconnect the apps you connected in the dashboard (**Connections → AI assistants**).
- No token can connect channels, manage members, create API keys or delete a workspace.

## Policies

- Privacy policy: https://replybuddy.io/legal/en/PrivacyPolicy.html
- Data processing policy: https://replybuddy.io/legal/en/DataProcessingPolicy.html
- Terms of service: https://replybuddy.io/legal/en/TermsOfService.html
- Support: support@replybuddy.io
