---
sidebar_position: 2
title: Widgets
---

# Widgets API

You can manage donation and payment widgets with the API. Widgets set the appearance and behavior of the embedded payment forms.

## List Widgets

```
GET /api/v1/widgets
```

The response contains all widgets that belong to the authenticated user.

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

The response contains one widget, selected by ID.

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

This request creates a widget. The response contains the ID of the new widget.

**Request body:**

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `name` | string | Yes | *required* | Name of the widget for display |
| `mode` | string | No | `"donation"` | `"donation"` (open amount) or `"payment"` (fixed amount) |
| `theme` | string | No | `"system"` | `"system"`, `"light"`, or `"dark"` |
| `show_stats` | boolean | No | `true` | Show the total received and the transaction count on the widget |
| `preset_amounts` | number[] | No | `[]` | XMR amounts that appear as quick-select buttons |
| `button_text` | string | No | `"Donate"` | Text on the submit button |
| `custom_message` | string | No | `"Your support is appreciated!"` | Message above the form |
| `thank_you_message` | string | No | `"Thank you for your donation!"` | Message after confirmation |
| `primary_color` | string | No | `"#ff6600"` | Hex color for the widget accent |
| `redirect_url` | string | No | `""` | URL for the redirect after confirmation (payment mode) |

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

This request updates some fields of a widget. Include only the fields that you want to change.

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

This request deletes a widget. The `widget_id` of each existing transaction that links to this widget becomes `NULL`. The transactions remain.

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
