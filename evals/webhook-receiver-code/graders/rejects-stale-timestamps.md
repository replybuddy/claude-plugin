---
type: llm
focus: {source: file, path: webhook_receiver.py}
---
PASS if the code rejects a request whose t is too far from the current time, about 5 minutes.
FAIL if t is used only for the signature and an old signed request is still accepted.
