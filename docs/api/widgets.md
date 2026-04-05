---
sidebar_position: 2
title: Widgets
---

# Widgets API

Manage donation and payment widgets programmatically. Widgets define the appearance and behavior of the embeddable payment forms.

## List Widgets

```
GET /api/v1/widgets
```

Returns all widgets belonging to the authenticated user.

**Example:**

```bash
curl -H "Authorization: Bearer tip_<your_api_key>" \
  https://your-instance.example.com/api/v1/widgets
```

**Response `200 OK`:**

```json
{
  "data": [
    {
      "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
      "name": "Project Donations",
      "mode": "donation",
      "preset_amounts": [0.01, 0.05, 0.1],
      "button_text": "Support This Project",
      "custom_message": "Thank you for your support!",
      "thank_you_message": "Thank you for your donation!",
      "primary_color": "#ff6600",
      "theme": "dark",
      "show_stats": true,
      "redirect_url": "",
      "created_at": 1700000000,
      "updated_at": 1700000000
    }
  ]
}
```

## Get Widget

```
GET /api/v1/widgets/{id}
```

Returns a single widget by ID.

**Example:**

```bash
curl -H "Authorization: Bearer tip_<your_api_key>" \
  https://your-instance.example.com/api/v1/widgets/f47ac10b-58cc-4372-a567-0e02b2c3d479
```

**Response `200 OK`:**

```json
{
  "data": {
    "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
    "name": "Project Donations",
    "mode": "donation",
    "preset_amounts": [0.01, 0.05, 0.1],
    "button_text": "Support This Project",
    "custom_message": "Thank you for your support!",
    "thank_you_message": "Thank you for your donation!",
    "primary_color": "#ff6600",
    "theme": "dark",
    "show_stats": true,
    "redirect_url": "",
    "created_at": 1700000000,
    "updated_at": 1700000000
  }
}
```

**Error `404 Not Found`:**

```json
{"error": "widget not found"}
```

## Create Widget

```
POST /api/v1/widgets
```

Creates a new widget. Returns the ID of the created widget.

**Request body:**

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `name` | string | Yes | *required* | Display name for the widget |
| `mode` | string | No | `"donation"` | `"donation"` (open amount) or `"payment"` (fixed amount) |
| `theme` | string | No | `"system"` | `"system"`, `"light"`, or `"dark"` |
| `show_stats` | boolean | No | `true` | Show total received and transaction count on widget |
| `preset_amounts` | number[] | No | `[]` | Preset XMR amounts shown as quick-select buttons |
| `button_text` | string | No | `"Donate"` | Text on the submit button |
| `custom_message` | string | No | `"Your support is appreciated!"` | Message shown above the form |
| `thank_you_message` | string | No | `"Thank you for your donation!"` | Message shown after confirmation |
| `primary_color` | string | No | `"#ff6600"` | Hex color for the widget accent |
| `redirect_url` | string | No | `""` | URL to redirect to after confirmation (payment mode) |

**Example:**

```bash
curl -X POST \
  -H "Authorization: Bearer tip_<your_api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Premium Access",
    "mode": "payment",
    "theme": "dark",
    "preset_amounts": [0.5],
    "button_text": "Pay 0.5 XMR",
    "redirect_url": "https://example.com/thank-you"
  }' \
  https://your-instance.example.com/api/v1/widgets
```

**Response `201 Created`:**

```json
{"id": "b3d7e9a1-4c2f-4e8b-9f1a-2d3c4b5a6e7f"}
```

## Update Widget

```
PATCH /api/v1/widgets/{id}
```

Partially updates a widget. Only include the fields you want to change.

**Example:**

```bash
curl -X PATCH \
  -H "Authorization: Bearer tip_<your_api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Updated Widget Name",
    "button_text": "Donate XMR",
    "preset_amounts": [0.01, 0.05, 0.1, 0.5]
  }' \
  https://your-instance.example.com/api/v1/widgets/f47ac10b-58cc-4372-a567-0e02b2c3d479
```

**Response `200 OK`:**

```json
{
  "data": {
    "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
    "name": "Updated Widget Name",
    "mode": "donation",
    "preset_amounts": [0.01, 0.05, 0.1, 0.5],
    "button_text": "Donate XMR",
    "custom_message": "Thank you for your support!",
    "thank_you_message": "Thank you for your donation!",
    "primary_color": "#ff6600",
    "theme": "dark",
    "show_stats": true,
    "redirect_url": "",
    "created_at": 1700000000,
    "updated_at": 1700003600
  }
}
```

## Delete Widget

```
DELETE /api/v1/widgets/{id}
```

Deletes a widget. Existing transactions linked to this widget will have their `widget_id` set to `NULL` (transactions are preserved).

**Example:**

```bash
curl -X DELETE \
  -H "Authorization: Bearer tip_<your_api_key>" \
  https://your-instance.example.com/api/v1/widgets/f47ac10b-58cc-4372-a567-0e02b2c3d479
```

**Response `200 OK`:**

```json
{"deleted": "f47ac10b-58cc-4372-a567-0e02b2c3d479"}
```
