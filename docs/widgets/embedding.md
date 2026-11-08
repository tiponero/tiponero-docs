---
sidebar_position: 1
title: Embedding Widgets
---

# Embedding Widgets

A Tiponero widget is a self-contained HTML page. You can embed a widget on any website with an iframe. Each widget has a unique URL that contains its ID.

## Widget URL

```
https://your-instance.example.com/widget/<widget-id>
```

Widgets are public. A visitor does not need authentication to view or use a widget.

## iframe Embed

Add a widget to any HTML page:

```html
<iframe
  src="https://your-instance.example.com/widget/f47ac10b-58cc-4372-a567-0e02b2c3d479"
  width="400"
  height="600"
  frameborder="0"
  style="border: none; border-radius: 8px;"
  allow="clipboard-write"
></iframe>
```

### Recommended Dimensions

| Context | Width | Height |
|---|---|---|
| Sidebar embed | `350px` | `550px` |
| Centered on page | `400px` | `600px` |
| Full-width mobile | `100%` | `600px` |

The widget adapts to smaller widths. Tiponero recommends a minimum width of `300px`.

### Responsive Embed

For an embed that scales with its container, use this code:

```html
<div style="max-width: 400px; margin: 0 auto;">
  <iframe
    src="https://your-instance.example.com/widget/f47ac10b-58cc-4372-a567-0e02b2c3d479"
    width="100%"
    height="600"
    frameborder="0"
    style="border: none; border-radius: 8px;"
    allow="clipboard-write"
  ></iframe>
</div>
```

## Widget Appearance

Widgets show:

1. **Owner profile:** avatar, display name, and bio (from user settings)
2. **Custom message:** configurable per widget
3. **Preset amount buttons:** quick-select XMR amounts (configurable)
4. **Custom amount input:** manual XMR amount entry
5. **Donor name and note:** optional fields that let the payer identify themselves
6. **Submit button:** configurable button text
7. **Stats:** total received and transaction count (can be disabled per widget)

## Theme Options

Each widget uses one of three themes:

| Theme | Behavior |
|---|---|
| `system` | Matches the dark or light mode of the visitor's operating system |
| `light` | Always light background |
| `dark` | Always dark background |

Set the theme when you create or update a widget in the API or in the admin panel.

## Customization

| Property | Default | Description |
|---|---|---|
| `button_text` | `"Donate"` | Text on the submit button |
| `custom_message` | `"Your support is appreciated!"` | Message above the form |
| `thank_you_message` | `"Thank you for your donation!"` | Message that the widget shows after the confirmation |
| `primary_color` | `"#ff6600"` | Accent color (hex) |
| `preset_amounts` | `[]` | Array of XMR amounts for quick-select buttons |
| `show_stats` | `true` | Show total received and transaction count |
| `redirect_url` | `""` | Post-confirmation redirect (payment mode only) |

## Cross-Origin Considerations

Tiponero serves the widgets from your instance. If your website uses a different domain, the iframe still works and needs no CORS configuration. An iframe does not need CORS headers.

The `allow="clipboard-write"` attribute lets the widget copy the Monero address to the clipboard when the visitor clicks the address.
