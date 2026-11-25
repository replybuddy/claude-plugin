# Evals

Three cases, each with its own mocks of the ReplyBuddy MCP server (`<case>/mocks/replybuddy/<tool>.md`), so no real workspace or sign-in is needed. Tool schemas (`_tools.json`) and the shape of each answer are copied from the live server, and `get_guide.md` is its webhooks section, abridged; workspace data in the mocks is made up.

A mock answers the same way for any input, so a wrong id gets a normal answer instead of a 404, and graders check the id. Mocks do not serve the server's instructions.

| Case | Checks |
|---|---|
| `diagnose-silent-conversation` | Finds that a dialog is silent because it was switched to an operator, and changes nothing |
| `webhook-for-deal-moves` | Creates the endpoint with `create_webhook_endpoint` for `deal.moved` only, sends a test delivery and explains signature checks |
| `webhook-receiver-code` | Writes a FastAPI receiver: route of the registered endpoint, signature over the raw body, stale timestamps, deduplication on the event id, answer before slow work, fields from `data` |

## Run

Run from a checkout that holds only this plugin: the CLI scans the whole directory, so git worktrees under `.worktrees/` add extra copies of the plugin. `webhook-receiver-code` writes a file, so grant Write.

```bash
claude plugin eval . --runs 3 --allow-tools Write
```

The default baseline arm runs without the plugin, which also removes the MCP server, so the delta mostly shows "server vs. no server". To measure what the skills add on top of the server, run the same suite on a copy without `skills/` and compare the scores:

```bash
rm -rf /tmp/replybuddy-mcp-only && cp -R . /tmp/replybuddy-mcp-only && rm -rf /tmp/replybuddy-mcp-only/skills
claude plugin eval /tmp/replybuddy-mcp-only --runs 3 --ablation none --allow-tools Write
```

Results go to `evals/results/` (ignored by git). `plugin eval` needs a recent Claude Code; 2.1.119 does not have it, 2.1.309 does.
