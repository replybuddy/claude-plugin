---
type: llm
---
PASS if the reply does all of: (1) tells the user the signing secret is shown only once and must be stored now in a secret store or environment variable; (2) explains verification as HMAC-SHA256 with the secret over "<t>.<raw body>" compared with v1 from the X-ReplyBuddy-Signature header; (3) mentions deduplicating on the event id or rejecting old timestamps; (4) reports the result of the test delivery.
FAIL if any of the four is missing, or the reply says the signature is computed over parsed or re-serialized JSON.
