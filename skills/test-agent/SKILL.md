---
name: test-agent
description: Use when someone wants to see how their ReplyBuddy AI agent answers before real clients do, or to make sure a change did not break it — "try the bot as a parent asking about prices", "check that it asks for the phone number", "run our test cases", "add a regression test for this answer", "did my new prompt make things worse". Covers sandbox dialogs and saved agent test cases with expected answers.
---

# Test a ReplyBuddy agent

Two tools for two jobs:

- **Sandbox**: one private dialog per workspace with the agent, using the live configuration. Nothing reaches messengers or real clients. Good for walking a scenario step by step.
- **Test cases**: saved question + expected answer pairs. A run asks every question in a fresh context and judges the answer against the expectation. Good for regressions after every change.

## Sandbox dialog

1. Plan the scenario with the user: who the client is, what they write, what the agent must do at each step (which goal, which data to collect, when to hand over).
2. `sandbox_list_messages` shows the current sandbox history (always the whole dialog). Earlier messages influence the agent: a new scenario should start from an empty sandbox, so ask the user before `sandbox_reset` (it clears the sandbox dialog only, but the history is lost). For a check of one question, a test case is cleaner than a sandbox with old history.
3. `sandbox_send_message` with the client's text. The reply is generated asynchronously: wait about 5–10 seconds, call `sandbox_list_messages`, and repeat until a message with `role: "ai"` appears after yours. A reply usually takes 5–20 seconds; the agent may answer with several messages.
4. Read each `ai` message's `content` and `thinking`. `thinking` shows the agent's reasoning and what it did: knowledge it loaded, tags it set, data fields it collected, tools it called.
5. Continue the dialog as the client would. To get another version of the last reply, `api_create /agents/{id}/sandbox/regenerate`.
6. Report per step: what the client wrote, what the agent answered, whether it matched the expectation, and what in the configuration explains a miss (identity, a behavior section, a goal's `instruction` or `stop_condition`, a knowledge entry).

A good answer worth keeping becomes a test case with `api_create /agents/{id}/sandbox/messages/{msg_id}/save-as-test` (the `msg_id` of the `ai` message). It stores the reply word for word as `expected_answer`; rewrite that as criteria with `api_update /agent-tests/{id}` (see below).

In `/agents/{id}/…` paths any id works: the server uses the workspace of the call (its id is `agent_id` in `get_workspace_snapshot`).

## Test cases

1. `list_agent_tests` — existing cases, their last `status` (`passed`, `failed`) and `actual_answer`.
2. `create_agent_test` with `question` (what the client writes) and `expected_answer`. Write the expectation as criteria, not as an exact text: "Gives the price range 5.5–9.9k, offers the free diagnostic, asks for the parent's phone; does not promise a grade without conditions." Exact wording fails on every harmless rephrase.
3. `run_agent_tests` runs every case and returns a `job_id`. Poll `get_agent_test_job` with it every 10–20 seconds until `completed` equals `total`. To run one case: `api_create /agent-tests/{id}/run`.
4. For each `failed` case, compare `actual_answer` with `expected_answer` and say whether the agent or the expectation is wrong. Edit a case with `api_update /agent-tests/{id}`; delete one with `api_delete /agent-tests/{id}`.

## After a configuration change

1. `get_workspace_snapshot` — confirm the runtime sees the change.
2. Rerun the sandbox steps the change was meant to fix.
3. `run_agent_tests` — make sure the other cases still pass.
4. Report what passed, what failed and why, separately from what you did not test.

## Pitfalls

- Each test case is one client message answered from the start goal with no dialog history; scenarios that need several turns belong in the sandbox.
- In the sandbox, goal switches, collected data fields, tags, notes and catalog skill actions (such as booking a meeting) are simulated: `thinking` marks them as simulated and no data changes. Calls of enabled `custom_api` skills are real requests to the business API.
- There is no draft mode: the sandbox and real dialogs use the same saved configuration, so a change reaches real clients as soon as it is saved. Test right after saving and be ready to revert (`list_history` shows the previous versions).
