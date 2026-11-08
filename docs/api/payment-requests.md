---
sidebar_position: 8
title: Payment Requests
---

# Payment Requests

`POST /api/v1/transactions` creates a payment request. It returns a subaddress, the amount, a `monero:` URI, and a hosted payment page.

A widget is optional. Without one, the request is standalone and uses `/pay/<id>`.

## Create a request

```
POST /api/v1/transactions
```

**Body:**

| Field | Type | Description |
|---|---|---|
| `amount_xmr` | number | Amount in XMR. Use this or `amount_fiat`, not both |
| `amount_fiat` | number | Amount in fiat. The XMR amount is computed at the current rate |
| `currency` | string | Fiat currency for `amount_fiat`. Defaults to `FIAT_CURRENCY` |
| `widget_id` | string | Optional. The widget supplies defaults and the payment page style |
| `description` | string | Merchant-facing label, also shown on the payment page |
| `donor_name` | string | Optional payer name |
| `note` | string | Optional payer note |
| `redirect_url` | string | Send the payer here after confirmation |
| `metadata` | object | Your own JSON, returned in webhooks. Maximum 4 KiB |
| `expires_in` | integer | Seconds until the request closes. 60 to 2592000 |
| `confirmations` | integer | `0` for 0-conf, `1` to `100`. Omit to use the widget or global default |
| `is_test` | boolean | Marks the row as test data. Excluded from stats and export |

If you omit the amount and pass a payment widget, the widget price is used. A donation widget or a standalone request may omit the amount to create an open-ended request.

**Example:**

```bash
curl -X POST https://tiponero.example/api/v1/transactions \
  -H "Authorization: Bearer tip_<your_api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "amount_fiat": 12.50,
    "currency": "USD",
    "description": "Order #1042",
    "confirmations": 0,
    "expires_in": 1800,
    "redirect_url": "https://shop.example/thanks",
    "metadata": {"order_id": "1042"}
  }'
```

**Response `201 Created`:**

```json
{
  "data": {
    "id": "c65b7e91-a54d-4e63-81e4-1ddc377c7bdb",
    "widget_id": "",
    "status": "pending",
    "subaddress": "8...",
    "amount": 8267742576,
    "amount_xmr": "0.008267742576",
    "received_amount": 0,
    "received_amount_xmr": "0.000000000000",
    "remaining_amount": 8267742576,
    "remaining_amount_xmr": "0.008267742576",
    "fiat_amount": 12.5,
    "fiat_currency": "USD",
    "description": "Order #1042",
    "confirmations": 0,
    "confirmations_required": 0,
    "is_payment": true,
    "is_test": false,
    "created_at": 1794161339,
    "expires_at": 1794163139,
    "monero_uri": "monero:8...?tx_amount=0.008267742576",
    "payment_url": "https://tiponero.example/pay/c65b7e91-...",
    "qr_url": "https://tiponero.example/pay/c65b7e91-.../qr",
    "status_url": "https://tiponero.example/pay/c65b7e91-.../status"
  }
}
```

Send the payer to `payment_url`. Poll `status_url` if you render your own checkout.

## Amount rules

- Send `amount_xmr` or `amount_fiat`. Sending both returns `422`.
- A fiat amount locks the rate at creation. The response records both the fiat amount and the piconero amount.
- A widget in payment mode ignores the browser form amount, but the API can set one. An API key is a trusted caller.

## What happens next

1. Tiponero calls `create_address` and stores the subaddress.
2. The payment monitor polls `get_transfers` and updates the row.
3. Each state change fires the webhooks and channels subscribed to that event.
4. A partial payment keeps the request open. The payment page shows the remaining amount and a new QR code.
5. At expiry the request becomes `expired`. A partial payment stays in `received_amount`.

## Status values

`pending`, `mempool`, `confirming`, `partial`, `confirmed`, `reverted`, `expired`.

`reverted` means a 0-conf payment left the mempool. The request stays under watch for two hours and returns to `confirmed` if the transfer reappears.
