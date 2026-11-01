---
type: llm
focus: {source: file, path: webhook_receiver.py}
---
PASS if the handler answers 2xx without waiting for process_deal_moved, for example through a background task or a queue.
FAIL if the response waits for the call, because it takes about a minute and the sender gives up after 10 seconds.
