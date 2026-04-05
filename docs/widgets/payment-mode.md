---
sidebar_position: 4
title: Payment Mode
---

# Payment Mode

Payment mode creates fixed-amount payment requests with automatic redirect on confirmation. Use this for selling digital goods, services, subscriptions, or any scenario where the amount is predetermined.

## How It Works

1. Visitor opens the widget
2. The fixed amount is displayed (no custom amount input)
3. Visitor clicks the pay button
4. A unique Monero subaddress is generated via `monero-wallet-rpc`
5. A QR code is displayed with a `monero:` URI containing the address and amount
6. The widget polls for payment status every 5 seconds via HTMX
7. Once confirmed, the visitor is **automatically redirected** to the configured `redirect_url`

## Creating a Payment Widget

### Via Admin Panel

1. Navigate to **Widgets** and click **Create Widget**
2. Set the mode to **Payment**
3. Set the fixed amount in the preset amounts field
4. Enter a `redirect_url` for post-confirmation redirect
5. Save

### Via API

```bash
curl -X POST \
  -H "Authorization: Bearer tip_<your_api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Premium Access - 0.5 XMR",
    "mode": "payment",
    "preset_amounts": [0.5],
    "button_text": "Pay 0.5 XMR",
    "custom_message": "Complete your purchase",
    "thank_you_message": "Payment confirmed! Redirecting...",
    "redirect_url": "https://example.com/thank-you?session=abc123",
    "primary_color": "#ff6600",
    "theme": "dark"
  }' \
  https://your-instance.example.com/api/v1/widgets
```

## Redirect URL

The `redirect_url` is where the visitor is sent after payment confirmation. This is set via the `HX-Redirect` header in the HTMX status poll response.

You can include query parameters in the redirect URL to pass context back to your application:

```
https://example.com/thank-you?session=abc123&plan=premium
```

:::tip
If you need to verify the payment on your backend, query the [Transactions API](/docs/api/transactions) to confirm the transaction status before granting access.
:::

## Differences from Donation Mode

| Feature | Donation Mode | Payment Mode |
|---|---|---|
| Amount | Open (user chooses) | Fixed (preset) |
| Custom amount input | Yes | No |
| After confirmation | Thank-you message | Redirect to `redirect_url` |
| `is_payment` field | `false` | `true` |
| Typical use case | Tips, donations, funding | Product purchases, access tokens |

## Integration Pattern

A typical payment integration flow:

1. Your application creates a payment widget (or reuses an existing one)
2. Embed the widget on your checkout page
3. Visitor completes payment through the widget
4. Visitor is redirected to your `redirect_url`
5. Your backend queries `GET /api/v1/transactions?status=confirmed` to verify payment
6. Grant access or deliver the product

For automated verification, poll the transactions API on your backend or implement a webhook receiver (not yet built into Tiponero: check the database directly or use the API).
