# Evals

Three cases, each with its own mocks of the ReplyBuddy MCP server (`<case>/mocks/replybuddy/<tool>.md`), so no real workspace or sign-in is needed. Tool schemas (`_tools.json`) and `search_api` / `describe_api` answers are recorded from the live server; workspace data in the mocks is made up.

Mocks answer the same way for any path, because the CLI substitutes `{input.x}` into a fixture name only as a single segment without slashes. A wrong path therefore gets a normal answer instead of a 404, and graders check the path. Mocks do not serve the server's instructions.

| Case | Checks |
|---|---|
| `diagnose-silent-conversation` | Finds that a dialog is silent because it was switched to an operator, and changes nothing |
| `set-start-goal` | Reads the goal graph before `replace_goal_graph` and keeps the existing edges |
| `webhook-for-deal-moves` | Creates the endpoint with `create_webhook_endpoint` for `deal.moved` only, sends a test delivery and explains signature checks |

## Run

```bash
claude plugin eval . --runs 3
```

The default baseline arm runs without the plugin, which also removes the MCP server, so the delta mostly shows "server vs. no server". To measure what the skills add on top of the server, run the same suite on a copy without `skills/` and compare the scores:

```bash
rm -rf /tmp/replybuddy-mcp-only && cp -R . /tmp/replybuddy-mcp-only && rm -rf /tmp/replybuddy-mcp-only/skills
claude plugin eval /tmp/replybuddy-mcp-only --runs 3 --ablation none
```

Results go to `evals/results/` (ignored by git). `plugin eval` needs a recent Claude Code; 2.1.119 does not have it, 2.1.309 does.
