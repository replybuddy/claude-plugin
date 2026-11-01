# ReplyBuddy plugin for Claude

[ReplyBuddy](https://replybuddy.io) is an AI agent that answers clients in messengers (Telegram, Instagram, VK, Avito, MAX, Zalo) and has a built-in CRM. This plugin lets you set up, test and debug that agent from Claude, and connect it to your own backend, without clicking through the dashboard.

It adds the ReplyBuddy MCP server (`https://mcp.replybuddy.io/mcp`) and five skills that teach Claude how to use it well.

## What you can ask

- "Set up our agent for the spring course: here is the price list and the FAQ. It should collect the parent's phone number and hand over to a manager."
- "Why did the bot tell this client the wrong price yesterday?"
- "Add a stage 'Invoice sent' after 'Qualified' and create a call task for the deal owner when a deal gets there."
- "When a client asks about their order, the agent should check it in our API at `https://api.example.com/orders/{id}`."
- "Send new client messages and deal changes to our backend, and write the endpoint that verifies the signature."
- "Run our test cases and tell me what broke after today's change."

## Skills

| Skill | When Claude uses it |
|---|---|
| `setup-sales-agent` | Setting up or reworking the agent: identity, behavior, knowledge, QA answers, goals with data to collect and reminders, catalog skills |
| `connect-backend` | Linking ReplyBuddy with your system: `custom_api` skills (the agent calls your API), signed webhooks (ReplyBuddy calls your backend), API keys (your backend calls ReplyBuddy) |
| `configure-crm` | Deal funnels, stages and their automations, deal fields, task types |
| `test-agent` | Sandbox dialogs and saved test cases, before real clients see a change |
| `diagnose-agent-reply` | Finding out why the agent answered, stayed silent or acted the way it did in a real dialog |

Claude picks the skill from your request; you can also call one directly, for example `/replybuddy:test-agent`.

## Install

You need a ReplyBuddy account with access to a workspace. Setup changes need the owner or admin role; members with other roles can read and work with the inbox and CRM within their permissions.

**Claude Code**

```
/plugin marketplace add replybuddy/replybuddy-claude-plugin
/plugin install replybuddy@replybuddy
```

Then run `/mcp`, choose `plugin:replybuddy:replybuddy` and sign in. On the ReplyBuddy consent screen pick the workspace and the access level.

**claude.ai and Claude Desktop**

Install the plugin from the directory. Until it is listed there, add the connector `https://mcp.replybuddy.io/mcp` under Settings → Connectors and upload a zip of this repository under Customize → Plugins for the skills.

**Without the plugin**

Any MCP client can connect to `https://mcp.replybuddy.io/mcp` directly. For a connection that can only read, use `https://mcp.replybuddy.io/mcp/readonly`. For CI and headless agents, use an API key instead of a browser sign-in: see [references/api-keys.md](references/api-keys.md).

## How it works

- One connection is bound to one workspace, chosen at sign-in. Claude never passes workspace ids.
- About 65 typed tools cover agent setup, the inbox, contacts, the CRM and testing. Every other operation of the ReplyBuddy REST API is reachable through `search_api` → `describe_api` → `api_read` / `api_create` / `api_update` / `api_delete`. The API reference is `https://mcp.replybuddy.io/openapi.json`.
- Read tools run without asking. Claude asks before every destructive or outward-facing action: sending messages and broadcasts, replacing whole sets, deleting, creating a webhook endpoint, sending a test request to your API.
- Connecting messenger channels, managing members, creating API keys and deleting a workspace are possible only in the ReplyBuddy dashboard.

Details: [workspace model](references/workspace-model.md), [tools](references/tools.md), [errors and limits](references/errors-and-limits.md), [webhooks](references/webhooks.md), [API keys](references/api-keys.md), [custom_api skills](references/custom-api-skills.md).

## Data: what is sent where

The plugin contains only Markdown instructions and the server address. It runs no code and sends nothing by itself.

- Claude talks only to `https://mcp.replybuddy.io` over HTTPS, using an OAuth token you grant (access token 1 hour, refresh token 30 days, revocable at any time).
- Tool results come from your workspace and may contain your clients' personal data: names, usernames, phone numbers, message texts, deal data. They become part of your Claude conversation.
- Changes go to ReplyBuddy only when Claude calls a write tool for your request.
- ReplyBuddy sends data outside only where you configure it: your `custom_api` URLs, your webhook URLs, and messages to clients when you ask Claude to send them.

Full table: [references/data-handling.md](references/data-handling.md). Privacy policy: https://replybuddy.io/legal/en/PrivacyPolicy.html. Terms: https://replybuddy.io/legal/en/TermsOfService.html.

## Support

support@replybuddy.io. Issues and suggestions for the skills: open an issue in this repository.

## License

MIT, see [LICENSE](LICENSE).
