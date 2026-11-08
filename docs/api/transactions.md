---
sidebar_position: 3
title: Transactions
---

# Transactions API

You can create payment requests and query transaction records. See [Payment Requests](./payment-requests.md) for creation. This page covers reading.

## List Transactions

```
GET /api/v1/transactions
```

The response contains a paginated list of transactions. You can add an optional status filter.

**Query parameters:**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `page` | integer | `1` | Page number (1-indexed) |
| `limit` | integer | `20` | Number of results per page (max 100) |
| `status` | string | *all* | Filter by status: `pending`, `mempool`, `confirming`, `partial`, `confirmed`, `reverted`, `expired` |

**Example:**

```bash
curl -H "Authorization: Bearer tip_<your_api_key>" \
  "https://your-instance.example.com/api/v1/transactions?status=confirmed&page=1&limit=10"
```

**Response `200 OK`:**

```json
{
  "data": [
    {
      "id": "c8a9b7e6-5d4f-3c2b-1a0e-9f8d7c6b5a4e",
      "widget_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
      "subaddress": "8A7Bf3...",
      "subaddress_index": 12,
      "amount": 50000000000,
      "amount_xmr": "0.05",
      "fiat_amount": 8.25,
      "fiat_currency": "USD",
      "status": "confirmed",
      "confirmations": 10,
      "is_payment": false,
      "donor_name": "Alice",
      "note": "Great project!",
      "tx_hash": "7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8",
      "created_at": 1700000000,
      "updated_at": 1700003600,
      "confirmed_at": 1700003600,
      "expires_at": 1700003600
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 1,
    "total_pages": 1
  }
}
```

**Response with no results:**

```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 0,
    "total_pages": 0
  }
}
```

## Get Transaction

```
GET /api/v1/transactions/{id}
```

The response contains one transaction, selected by ID.

**Example:**

```bash
curl -H "Authorization: Bearer tip_<your_api_key>" \
  https://your-instance.example.com/api/v1/transactions/c8a9b7e6-5d4f-3c2b-1a0e-9f8d7c6b5a4e
```

**Response `200 OK`:**

```json
{
  "data": {
    "id": "c8a9b7e6-5d4f-3c2b-1a0e-9f8d7c6b5a4e",
    "widget_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
    "subaddress": "8A7Bf3...",
    "subaddress_index": 12,
    "amount": 50000000000,
    "amount_xmr": "0.05",
    "fiat_amount": 8.25,
    "fiat_currency": "USD",
    "status": "confirmed",
    "confirmations": 10,
    "is_payment": false,
    "donor_name": "Alice",
    "note": "Great project!",
    "tx_hash": "7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8",
    "created_at": 1700000000,
    "updated_at": 1700003600,
    "confirmed_at": 1700003600,
    "expires_at": 1700003600
  }
}
```

**Error `404 Not Found`:**

```json
{"error": "transaction not found"}
```

## Response Fields

| Field | Type | Description |
|---|---|---|
| `id` | string | UUID of the transaction |
| `widget_id` | string\|null | UUID of the widget that made this transaction (`null` if the widget is deleted) |
| `subaddress` | string | Unique Monero subaddress that belongs to this transaction |
| `subaddress_index` | integer | Index of the subaddress in the wallet |
| `amount` | integer | Amount in piconero (1 XMR = 1,000,000,000,000) |
| `amount_xmr` | string | Amount as an XMR string |
| `fiat_amount` | number\|null | Fiat equivalent when Tiponero creates the transaction |
| `fiat_currency` | string\|null | ISO 4217 currency code (for example, `"USD"`) |
| `status` | string | `pending`, `mempool`, `confirming`, `partial`, `confirmed`, `reverted`, or `expired` |
| `confirmations` | integer | Current number of block confirmations |
| `is_payment` | boolean | `false` for donations, `true` for fixed-amount payments |
| `donor_name` | string\|null | Optional name from the donor |
| `note` | string\|null | Optional message from the donor |
| `tx_hash` | string\|null | Monero transaction hash (available after confirmation) |
| `created_at` | integer | Unix timestamp of creation |
| `updated_at` | integer | Unix timestamp of last status update |
| `confirmed_at` | integer\|null | Unix timestamp of confirmation |
| `expires_at` | integer | Unix timestamp when the transaction expires |
