# Webhooks

Organization events can be delivered to an HTTPS endpoint you register.

The signed payload is HMAC-SHA256 over `{unix_seconds}.{body}`. Headers:

```text
Thirp-Timestamp
Thirp-Signature     t=...,v1=...
Thirp-Event-Id
Idempotency-Key     same as the event id
```

Replay window is 5 minutes. The webhook secret is shown once on create; later GET responses omit it.

Event bodies are metadata (actor, action, target). They do not include application payloads or credential material.

List, create, disable, and delivery history are on the public `/v1` API. See [openapi/v1.yaml](../openapi/v1.yaml).
