# Testing the agent: sandbox and test cases

Check a change before real clients see it.

## Sandbox

The sandbox is a test conversation with the agent that real clients do not see.

- `sandbox_send_message` sends a message; the reply arrives asynchronously, read it with `sandbox_list_messages`.
- `get_sandbox_state` shows the sandbox conversation's current stage, collected data and reminders.
- `regenerate_sandbox_reply` replaces the agent's last reply with a new one; `sandbox_reset` starts the conversation over.
- `save_sandbox_reply_as_test` keeps a reply as a test case.

## Test cases

A test case is a client `question` with an `expected_answer`. A run fills `actual_answer` and sets `status` to `passed` or `failed`.

- `list_agent_tests`, `create_agent_test`, `update_agent_test`, `delete_agent_test`, `delete_agent_tests` (several by id).
- `run_agent_tests` runs them all and `run_agent_test` runs one; poll `get_agent_test_job` with the returned job id for the results.
