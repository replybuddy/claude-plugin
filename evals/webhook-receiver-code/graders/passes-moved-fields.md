---
type: llm
focus: {source: file, path: webhook_receiver.py}
---
PASS if process_deal_moved gets the deal id from data.deal, from_stage_id from data.from_stage_id and moved_by from data.moved_by, is called only for deal.moved, and other event types such as webhook.test are answered with 2xx.
FAIL if a field is read from the wrong place, or another event type gets an error answer or triggers the call.
