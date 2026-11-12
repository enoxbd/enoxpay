# Enox Pay — Webhook Integration Guide

This guide documents the HTTP headers, payload template placeholders, and HMAC-SHA256 verification protocol used by **Enox Pay** (`v1.0.0`).

---

## Request Headers

Every webhook delivery uses HTTP `POST` with `Content-Type: application/json; charset=utf-8`:

| Header | Description |
| --- | --- |
| `Accept` | `application/json` |
| `X-Enox-Event` | Event name (e.g., `transaction.captured`) |
| `Authorization` | `Bearer <bearerToken>` (or `<authKey>` if no bearer token is set) |
| `X-API-Key` | Sent alongside `Authorization` when both bearer token and API key are configured |
| `X-Enox-Timestamp` | Unix timestamp in seconds (present when Webhook Secret is configured) |
| `X-Enox-Signature` | `sha256=<hmac_hex>` (present when Webhook Secret is configured) |
| Custom Headers | Any `Header-Name: Value` lines defined in **Config** |

---

## Template Placeholders

You can customize the JSON body template inside **Config**. Available variables:

| Placeholder | Meaning |
| --- | --- |
| `{platform}` | Detected MFS / platform name (e.g., `bKash`, `Nagad`, `Rocket`, `Upay`) |
| `{amount}` | Parsed transaction amount |
| `{tx_id}` | Parsed transaction ID (`TrxID` / `TxnId`) |
| `{sender}` | Sender MSISDN or sender title |
| `{balance}` | Parsed post-transaction balance (if present in SMS) |
| `{fee}` | Parsed fee/charge (if present in SMS) |
| `{sim_slot}` | Receiving SIM slot identifier |
| `{source}` | Capture origin (`SMS`, `NOTIFICATION`, `INBOX`, or `MANUAL`) |
| `{sms}` | Full JSON-escaped message body |
| `{time}` | Epoch milliseconds when the event was captured |

---

## Verifying `X-Enox-Signature`

Enox Pay computes the HMAC-SHA256 over the exact string:

```text
signed_payload = "{X-Enox-Timestamp}.{raw_request_body}"
expected_header = "sha256=" + hmac_sha256_hex(webhook_secret, signed_payload)
```

Always validate both the signature (using constant-time comparison) and timestamp freshness (e.g., within 300 seconds) before crediting an account.

### Node.js (Express)

```js
import crypto from "node:crypto";

export function verifyEnoxWebhook(req, rawBody, secret) {
  const ts = req.header("X-Enox-Timestamp") || "";
  const sig = req.header("X-Enox-Signature") || "";
  if (!ts || !sig.startsWith("sha256=")) return false;

  const ageSec = Math.abs(Math.floor(Date.now() / 1000) - Number(ts));
  if (!Number.isFinite(ageSec) || ageSec > 300) return false;

  const expected =
    "sha256=" +
    crypto.createHmac("sha256", secret).update(`${ts}.${rawBody}`, "utf8").digest("hex");

  const a = Buffer.from(sig, "utf8");
  const b = Buffer.from(expected, "utf8");
  return a.length === b.length && crypto.timingSafeEqual(a, b);
}
```

### PHP

```php
<?php
function verifyEnoxWebhook(string $rawBody, string $secret): bool {
    $ts  = $_SERVER['HTTP_X_ENOX_TIMESTAMP'] ?? '';
    $sig = $_SERVER['HTTP_X_ENOX_SIGNATURE'] ?? '';
    if ($ts === '' || !str_starts_with($sig, 'sha256=')) {
        return false;
    }
    if (abs(time() - (int)$ts) > 300) {
        return false;
    }
    $expected = 'sha256=' . hash_hmac('sha256', $ts . '.' . $rawBody, $secret);
    return hash_equals($expected, $sig);
}
```

### Python

```python
import hashlib
import hmac
import time


def verify_enox_webhook(raw_body: str, timestamp: str, signature: str, secret: str) -> bool:
    if not timestamp or not signature.startswith("sha256="):
        return False
    try:
        if abs(int(time.time()) - int(timestamp)) > 300:
            return False
    except ValueError:
        return False

    digest = hmac.new(
        secret.encode("utf-8"),
        f"{timestamp}.{raw_body}".encode("utf-8"),
        hashlib.sha256,
    ).hexdigest()
    return hmac.compare_digest(f"sha256={digest}", signature)
```

---

## HTTP Status & Retries

- Return any **`2xx`** HTTP status code (e.g., `200 OK`) to mark the transaction as **`SUCCESS`**.
- Non-2xx responses or network timeouts mark the transaction as **`FAILED`** and automatically schedule retries via `WorkManager` up to the configured retry limit.
- Make your endpoint **idempotent** on `transaction_id` so retried deliveries never double-credit.
