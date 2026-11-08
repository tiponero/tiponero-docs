---
sidebar_position: 4
title: Payment Mode
---

# Payment Mode

Payment mode creates a payment request with a fixed amount. It also redirects the visitor after the confirmation. Use it to sell digital goods, services, or subscriptions. Use it also when you know the amount in advance.

## How It Works

1. Visitor opens the widget
2. The widget shows the fixed amount (there is no custom amount input)
3. Visitor clicks the pay button
4. The widget generates a unique Monero subaddress with `monero-wallet-rpc`
5. The widget shows a QR code with a `monero:` URI that contains the address and the amount
6. The widget polls the payment status every 5 seconds with HTMX
7. After the confirmation, the widget **automatically redirects** the visitor to the configured `redirect_url`

## Creating a Payment Widget

### Via Admin Panel

1. Navigate to **Widgets** and click **Create Widget**
2. Set the mode to **Payment**
3. Set the fixed amount in the preset amounts field
4. Enter a `redirect_url` for the redirect after the confirmation
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

The `redirect_url` is the address that the visitor goes to after the payment confirmation. The widget sets it with the `HX-Redirect` header in the HTMX status poll response.

You can include query parameters in the redirect URL to send context back to your application:

```
https://example.com/thank-you?session=abc123&plan=premium
```

:::tip
Make sure of the payment on your backend first. Query the [Transactions API](/docs/api/transactions) for the transaction status before you grant access.
:::

## Differences from Donation Mode

| Feature | Donation Mode | Payment Mode |
|---|---|---|
| Amount | Open (the user chooses) | Fixed (preset) |
| Custom amount input | Yes | No |
| After confirmation | Thank-you message | Redirect to `redirect_url` |
| `is_payment` field | `false` | `true` |
| Typical use case | Tips, donations, funding | Product purchases, access tokens |

## Integration Pattern

A typical payment integration flow:

1. Your application creates a payment widget (or reuses an existing one)
2. Embed the widget on your checkout page
3. Visitor completes payment through the widget
4. The widget redirects the visitor to your `redirect_url`
5. Your backend queries `GET /api/v1/transactions?status=confirmed` to make sure of the payment
6. Grant access or deliver the product

For automatic verification, poll the transactions API from your backend. You can also write a webhook receiver (Tiponero does not have one yet). Then read the database directly, or use the API.
