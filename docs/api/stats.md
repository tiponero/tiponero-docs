---
sidebar_position: 6
title: Stats
---

# Stats API

You can get aggregate statistics about transactions.

## Get Stats

```
GET /api/v1/stats
```

The response contains summary statistics for the transactions of the authenticated user.

**Example:**

```bash
curl -H "Authorization: Bearer tip_<your_api_key>" \
  https://your-instance.example.com/api/v1/stats
```

**Response `200 OK`:**

```json
{
  "data": {
    "total_transactions": 142,
    "total_amount": 5230000000000,
    "total_amount_xmr": "5.23",
    "pending_count": 3,
    "today_count": 7,
    "today_amount": 350000000000,
    "today_amount_xmr": "0.35"
  }
}
```

## Response Fields

| Field | Type | Description |
|---|---|---|
| `total_transactions` | integer | Total number of confirmed transactions |
| `total_amount` | integer | Total confirmed amount in piconero |
| `total_amount_xmr` | string | Total confirmed amount as an XMR string |
| `pending_count` | integer | Number of currently pending transactions |
| `today_count` | integer | Number of transactions created today |
| `today_amount` | integer | Total amount from transactions of today in piconero |
| `today_amount_xmr` | string | Total for today as an XMR string |

:::tip
The API always returns each amount in piconero (integer, machine-readable) and in XMR (string, human-readable). Use the piconero value for calculations. Use the XMR string for display.
:::
