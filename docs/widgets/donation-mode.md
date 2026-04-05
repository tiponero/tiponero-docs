---
sidebar_position: 3
title: Donation Mode
---

# Donation Mode

Donation mode allows visitors to send any amount of XMR. This is the default widget mode, designed for open-ended contributions like project funding, tips, or voluntary support.

## How It Works

1. Visitor opens the widget
2. Selects a preset amount or enters a custom XMR amount
3. Optionally enters their name and a note
4. Clicks the donate button
5. A unique Monero subaddress is generated via `monero-wallet-rpc`
6. A QR code is displayed with a `monero:` URI containing the address and amount
7. The widget polls for payment status every 5 seconds via HTMX
8. Once confirmed, a thank-you message is displayed

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

Preset amounts are displayed as quick-select buttons. Visitors can still enter a custom amount. The amounts are in XMR:

```json
"preset_amounts": [0.01, 0.05, 0.1, 0.5, 1.0]
```

If `preset_amounts` is empty or omitted, only the custom amount input is shown.

## Donor Information

The widget includes two optional fields for donors:

| Field | Description |
|---|---|
| **Name** | Displayed in the admin panel transaction list |
| **Note** | A message from the donor, visible in the admin panel |

Both fields are optional. They are stored in the `donor_name` and `note` columns of the transaction record.

## After Confirmation

In donation mode, when the payment reaches the required number of confirmations:

1. The widget displays the `thank_you_message`
2. No redirect occurs (unlike payment mode)
3. The transaction is marked `confirmed` in the database
