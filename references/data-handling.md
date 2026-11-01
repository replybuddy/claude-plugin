# What data goes where

The plugin itself contains only instructions (Markdown) and one server address (JSON). It runs no code, collects nothing and sends nothing on its own. All data moves through the ReplyBuddy MCP server, and only when Claude calls one of its tools.

## Connections

| From → to | What | When |
|---|---|---|
| Claude → `https://mcp.replybuddy.io` | OAuth sign-in, then MCP requests over HTTPS with a bearer token | Every tool call |
| ReplyBuddy → Claude | Tool results: workspace settings, knowledge, conversations and messages, contacts (names, usernames, phone numbers and other fields your clients shared), deals, tasks | When Claude reads them for your request |
| Claude → ReplyBuddy | The changes you ask for: settings, knowledge, QA, goals, skills, CRM records, messages to clients, broadcasts | Write and destructive tools; Claude asks before destructive ones |
| ReplyBuddy → your API | Requests from `custom_api` skills while the agent talks to clients, and one request per `test_custom_api_tool` call | Only to URLs you configured |
| ReplyBuddy → your webhook URL | Signed event POSTs, including the text of incoming client messages | Only for endpoints you created |
| ReplyBuddy → clients' messengers | Messages and broadcasts | Only for `send_message`, `resend_message`, `schedule_broadcast`, `send_broadcast`, and the agent's own replies |

What ReplyBuddy returns to Claude becomes part of your conversation with Claude and is handled under your agreement with Anthropic. Ask Claude for aggregates or a few records when you do not need full client data.

## Access

- The connection covers only the workspaces you tick on the consent screen, and acts with your role in each. The access level applies to all of them, and revoking the connection removes access to all of them. With several workspaces, Claude can see data from all of them in one conversation, including client messages: tick only the ones you need.
- On the consent screen you grant `read`, or `read` and `write`. Connect to `https://mcp.replybuddy.io/mcp/readonly` for a connection that can never change anything.
- Access tokens expire after 1 hour, refresh tokens after 30 days. You can disconnect the apps you connected in the dashboard (**Connections → API & MCP**).
- No token can connect channels, manage members, create API keys or delete a workspace.

## Policies

- Privacy policy: https://replybuddy.io/legal/en/PrivacyPolicy.html
- Data processing policy: https://replybuddy.io/legal/en/DataProcessingPolicy.html
- Terms of service: https://replybuddy.io/legal/en/TermsOfService.html
- Support: support@replybuddy.io
