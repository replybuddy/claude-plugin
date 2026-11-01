---
type: llm
---
PASS if the reply says the AI is silent because the conversation was switched to a human operator (status "manual" / assigned to an operator), and suggests handing it back to the AI (assign_conversation_ai or the inbox button) as a proposal.
FAIL if the reply blames the channel, the workspace reply mode or the agent's prompt, or does not name the manual status.
