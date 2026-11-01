---
type: llm
focus: {source: file, path: webhook_receiver.py}
---
PASS if repeated events are detected by the id field of the JSON body, so a repeated event does not call process_deal_moved again.
FAIL if there is no deduplication or the key is the X-ReplyBuddy-Delivery header.
