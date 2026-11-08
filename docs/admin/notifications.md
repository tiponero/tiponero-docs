---
sidebar_position: 4
title: Notifications
---

# Notifications

The **Notifications** page in the admin panel has two parts: webhooks for your own server, and channels for people.

## Events

Both kinds of endpoint subscribe to individual events:

| Event | When it fires |
|---|---|
| `payment.created` | A payment request was created |
| `payment.mempool` | A transfer covers the amount and is in the mempool |
| `payment.partial` | A transfer arrived but the total is below the request |
| `payment.confirming` | Confirmations are between 1 and the required count |
| `payment.confirmed` | The required confirmations are reached, or a 0-conf payment is seen |
| `payment.reverted` | A 0-conf payment left the mempool inside the two-hour window |
| `payment.expired` | The request closed with no funds |

## Webhooks

A webhook sends a JSON body to your URL. Create one in **Notifications > Webhooks**. Tiponero shows the signing secret once. Store it in your application.

### Request headers

| Header | Value |
|---|---|
| `X-Tiponero-Event` | The event name |
| `X-Tiponero-Delivery` | A unique delivery ID. Use it to discard duplicates |
| `X-Tiponero-Timestamp` | Unix seconds, included in the signature |
| `X-Tiponero-Signature` | `sha256=` + HMAC-SHA256 of `<timestamp>.<raw body>` |

### Body

```json
{
  "id": "9f2c...",
  "event": "payment.confirmed",
  "created_at": 1794161339,
  "data": {
    "transaction_id": "c65b7e91-...",
    "widget_id": "",
    "status": "confirmed",
    "subaddress": "8...",
    "amount": 10000000000,
    "amount_xmr": "0.01",
    "received_amount": 10000000000,
    "received_amount_xmr": "0.01",
    "remaining_amount": 0,
    "remaining_amount_xmr": "0",
    "confirmations": 0,
    "required_confirmations": 0,
    "is_payment": true,
    "is_test": false,
    "description": "Order #42",
    "tx_hash": "...",
    "created_at": 1794161339,
    "expires_at": 1794164939,
    "confirmed_at": 1794161400,
    "payment_url": "https://tiponero.example/pay/c65b7e91-..."
  }
}
```

### Verify the signature

```python
import hmac, hashlib

def verify(secret: str, timestamp: str, body: bytes, header: str) -> bool:
    expected = "sha256=" + hmac.new(
        secret.encode(), timestamp.encode() + b"." + body, hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, header)
```

Reject a timestamp older than a few minutes to block replay. Keep the delivery ID to discard duplicates.

### Retries

A failed delivery retries after 10 seconds, 1 minute, 5 minutes, 30 minutes, 2 hours, and 12 hours. After six attempts the delivery is marked failed. The delivery log shows the response code and the last error. Delivered rows are pruned after seven days.

### Secret storage

The signing secret is stored encrypted with AES-256-GCM, using a key derived from `ENCRYPTION_KEY`. It is not hashed: HMAC needs the raw value on every delivery. A stolen database file alone does not reveal it. Rotating the secret in the admin panel invalidates the old value immediately.

## Channels

| Channel | Required values |
|---|---|
| Telegram | Bot token and chat ID |
| Discord | Webhook URL |
| Slack | Incoming webhook URL |
| ntfy | Server (default `https://ntfy.sh`) and topic |
| WhatsApp | Twilio Account SID, auth token, sender, recipient |
| Email | SMTP host, port, user, password, from, to |
| Generic | Any URL that accepts a JSON body |

Email works with any SMTP provider. For SendGrid use host `smtp.sendgrid.net`, port `587`, user `apikey`, and an API key as the password.

WhatsApp needs a paid Twilio account, because Meta does not offer a direct webhook for messages. Every other channel is free.

Channel credentials are encrypted with the same key as webhook secrets. The edit form never displays a stored secret. Leave the field empty to keep the current value.

## Test a delivery

Each endpoint has a **Test** button. It queues a synthetic `webhook.test` event, so you can check the receiver before a real payment arrives.
