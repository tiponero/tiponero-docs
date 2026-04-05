---
sidebar_position: 1
title: Embedding Widgets
---

# Embedding Widgets

Tiponero widgets are self-contained HTML pages that can be embedded on any website using an iframe. Each widget has a unique URL based on its ID.

## Widget URL

```
https://your-instance.example.com/widget/<widget-id>
```

Widgets are public: no authentication is required for visitors to view and use them.

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

The widget is responsive and will adapt to smaller widths. The minimum recommended width is `300px`.

### Responsive Embed

For a responsive embed that scales with the container:

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

Widgets display:

1. **Owner profile:** avatar, display name, and bio (from user settings)
2. **Custom message:** configurable per widget
3. **Preset amount buttons:** quick-select XMR amounts (configurable)
4. **Custom amount input:** manual XMR amount entry
5. **Donor name and note:** optional fields for the payer to identify themselves
6. **Submit button:** configurable button text
7. **Stats:** total received and transaction count (can be disabled per widget)

## Theme Options

Each widget can use one of three themes:

| Theme | Behavior |
|---|---|
| `system` | Follows the visitor's OS dark/light mode preference |
| `light` | Always light background |
| `dark` | Always dark background |

Set the theme when creating or updating a widget via the API or admin panel.

## Customization

| Property | Default | Description |
|---|---|---|
| `button_text` | `"Donate"` | Text on the submit button |
| `custom_message` | `"Your support is appreciated!"` | Message above the form |
| `thank_you_message` | `"Thank you for your donation!"` | Message shown after confirmation |
| `primary_color` | `"#ff6600"` | Accent color (hex) |
| `preset_amounts` | `[]` | Array of XMR amounts for quick-select buttons |
| `show_stats` | `true` | Show total received and transaction count |
| `redirect_url` | `""` | Post-confirmation redirect (payment mode only) |

## Cross-Origin Considerations

Widgets are served from your Tiponero instance. If your website is on a different domain, the iframe works without any CORS configuration: iframes do not require CORS headers.

The `allow="clipboard-write"` attribute enables the widget to copy the Monero address to the visitor's clipboard when they click on it.
