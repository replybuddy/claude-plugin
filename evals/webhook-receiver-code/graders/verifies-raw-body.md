---
type: llm
focus: {source: file, path: webhook_receiver.py}
---
PASS if the code computes HMAC-SHA256 with the secret over "<t>.<raw body>", where the raw body is the request bytes read before any JSON parsing, takes t and v1 from X-ReplyBuddy-Signature and compares in constant time.
FAIL if the signature is computed over parsed or re-serialized JSON, the comparison uses ==, or the check can be skipped.
