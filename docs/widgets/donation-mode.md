---
sidebar_position: 3
title: Donation Mode
---

# Donation Mode

In donation mode, a visitor can send any amount of XMR. This is the default widget mode. Use it for open contributions, for example project funding, tips, or voluntary support.

## How It Works

1. Visitor opens the widget
2. Selects a preset amount or enters a custom XMR amount
3. Optionally enters their name and a note
4. Clicks the donate button
5. The widget generates a unique Monero subaddress with `monero-wallet-rpc`
6. The widget shows a QR code with a `monero:` URI that contains the address and the amount
7. The widget polls the payment status every 5 seconds with HTMX
8. After the confirmation, the widget shows a thank-you message

## Creating a Donation Widget

### Via Admin Panel

1. Navigate to **Widgets** and click **Create Widget**
2. Set the mode to **Donation**
3. Configure preset amounts, button text, and appearance
4. Save

### Via API

```bash
curl -X POST \
  -H "Authorization: Bearer tip_<your_api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Project Donations",
    "mode": "donation",
    "preset_amounts": [0.01, 0.05, 0.1],
    "button_text": "Donate",
    "custom_message": "Support this project with Monero",
    "thank_you_message": "Thank you for your generosity!",
    "primary_color": "#ff6600",
    "theme": "dark",
    "show_stats": true
  }' \
  https://your-instance.example.com/api/v1/widgets
```

## Preset Amounts

The widget shows the preset amounts as quick-select buttons. A visitor can also enter a custom amount. The amounts are in XMR:

```json
"preset_amounts": [0.01, 0.05, 0.1, 0.5, 1.0]
```

If `preset_amounts` is empty or omitted, the widget shows only the custom amount input.

## Donor Information

The widget has two optional fields for the donor:

| Field | Description |
|---|---|
| **Name** | Displayed in the admin panel transaction list |
| **Note** | A message from the donor, visible in the admin panel |

Both fields are optional. Tiponero stores them in the `donor_name` and `note` columns of the transaction record.

## After Confirmation

In donation mode, when the payment reaches the necessary number of confirmations:

1. The widget shows the `thank_you_message`
2. The widget does not redirect the visitor (this is different from payment mode)
3. Tiponero marks the transaction as `confirmed` in the database
