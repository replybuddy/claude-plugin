# Webhooks: ReplyBuddy calls your backend

A webhook endpoint receives a signed `POST` for each event it subscribed to. A workspace has up to 10 endpoints. Managing endpoints needs the owner or admin role.

## Events

| Type | When | `data` |
|---|---|---|
| `message.received` | A client sent a message (incoming only; the agent's and operators' messages do not fire it) | `conversation_id`, `message` |
| `conversation.handoff_requested` | The AI handed a dialog to a human | `conversation_id`, `reason` |
| `deal.created`, `deal.updated` | A deal was created or its fields changed | `deal` |
| `deal.moved` | A deal moved to another stage | `deal`, `from_stage_id`, `moved_by` |
| `deal.deleted` | A deal was deleted | `deal_id` |
| `webhook.test` | Sent only by the test call; cannot be subscribed to | `endpoint_id` |

Envelope:

```json
{"id": "5f0c7a2e-…", "type": "message.received", "api_version": "2026-10-30", "created_at": "2026-11-01T10:00:00Z", "agent_id": "…", "data": {}}
```

`id` is the event id. It is the same for every endpoint and every redelivery: deduplicate on it. The signing secret starts with `whsec_`.

## Request

- `POST`, `Content-Type: application/json`, `User-Agent: ReplyBuddy-Webhooks/1`
- `X-ReplyBuddy-Event`: the event type
- `X-ReplyBuddy-Delivery`: the delivery id, one per event and endpoint; retries and redeliveries keep it
- `X-ReplyBuddy-Signature: t=<unix seconds>,v1=<hex>` where `v1 = HMAC-SHA256(secret, "<t>.<raw request body>")`
- 10 second timeout; redirects are not followed, so a 3xx counts as a failure.

## Verifying the signature

1. Read the raw body bytes before any JSON parsing; re-serialized JSON will not match.
2. Split the header on `,` into `t` and `v1`.
3. Compute `HMAC-SHA256(secret, t + "." + raw_body)` as lowercase hex and compare it to `v1` in constant time.
4. Reject when `t` is more than 5 minutes away from your clock, to stop replays.
5. Answer `2xx` quickly and do the work asynchronously.

Python:

```python
import hmac, hashlib, time

def verify(secret: str, header: str, raw_body: bytes, tolerance: int = 300) -> bool:
    parts = dict(p.split("=", 1) for p in header.split(",") if "=" in p)
    t, sig = parts.get("t", ""), parts.get("v1", "")
    if not t.isdigit() or abs(time.time() - int(t)) > tolerance:
        return False
    expected = hmac.new(secret.encode(), f"{t}.".encode() + raw_body, hashlib.sha256).hexdigest()
    return hmac.compare_digest(expected, sig)
```

Node.js:

```js
import crypto from "node:crypto";

export function verify(secret, header, rawBody, tolerance = 300) {
  const parts = Object.fromEntries(header.split(",").map((p) => p.split("=", 2)));
  const t = Number(parts.t);
  if (!Number.isInteger(t) || Math.abs(Date.now() / 1000 - t) > tolerance) return false;
  const expected = crypto.createHmac("sha256", secret).update(`${parts.t}.`).update(rawBody).digest("hex");
  const sig = Buffer.from(parts.v1 ?? "", "utf8");
  return sig.length === expected.length && crypto.timingSafeEqual(sig, Buffer.from(expected, "utf8"));
}
```

Shell check of one delivery, with the body saved to `body.json`:

```bash
printf '%s.' "$T" | cat - body.json | openssl dgst -sha256 -hmac "$SECRET"
```

## Retries and auto-disable

- A non-2xx answer or a timeout is retried after 1 min, 5 min, 30 min, 2 h, 5 h, 10 h and 10 h: 8 attempts over about 28 hours, then the delivery is `failed`.
- When a delivery fails for good and nothing got through to the endpoint since, the endpoint is disabled (`enabled: false`, `disabled_reason`). New events wait. `PUT /webhook-endpoints/{id}` with `{"enabled": true}` resumes them.
- Delivery is at least once. An event may be lost only if the server crashes between saving the change and queueing the event.
- The delivery log keeps 30 days.

## Managing endpoints from Claude

| Task | Call |
|---|---|
| Create | `create_webhook_endpoint` with `url` (public `https`, cannot be changed later) and `events`. The response holds the signing `secret`, shown only once |
| List | `list_webhook_endpoints` |
| Change events, description, enable or disable | `update_webhook_endpoint` |
| Send a test event now | `send_test_webhook` (sent even to a disabled endpoint, no retries) |
| Rotate the secret | `rotate_webhook_secret` (the new secret is in the response only) |
| Delivery log | `list_webhook_deliveries` with `status` (`pending`, `delivered`, `failed`), `limit`, `offset`; each item has `attempts`, `last_status` (absent when no HTTP answer came), `last_error` and the exact `payload`. Request headers are not logged |
| Send a delivery again | `redeliver_webhook` (delivered or failed only; same event id) |
| Delete | `delete_webhook_endpoint` (also deletes its log) |

To move an endpoint to a new URL, create a new one, switch your backend, then delete the old one.
