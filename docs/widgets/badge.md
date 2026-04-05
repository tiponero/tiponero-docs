---
sidebar_position: 2
title: SVG Badge
---

# SVG Badge

Every widget has a live SVG badge endpoint that can be embedded in markdown documents, README files, or any website. The badge displays the widget name, a QR code linking to the widget, and optionally the total received amount and transaction count.

## Badge URL

```
https://your-instance.example.com/widget/<widget-id>/badge.svg
```

No authentication is required.

## Embedding in Markdown

Link the badge image to the widget URL so visitors can click through to donate:

```markdown
[![Donate with Monero](https://your-instance.example.com/widget/f47ac10b-58cc-4372-a567-0e02b2c3d479/badge.svg)](https://your-instance.example.com/widget/f47ac10b-58cc-4372-a567-0e02b2c3d479)
```

This renders as a clickable badge in GitHub READMEs, GitLab, and any markdown renderer that supports external images.

## Embedding in HTML

```html
<a href="https://your-instance.example.com/widget/f47ac10b-58cc-4372-a567-0e02b2c3d479">
  <img
    src="https://your-instance.example.com/widget/f47ac10b-58cc-4372-a567-0e02b2c3d479/badge.svg"
    alt="Donate with Monero"
    width="400"
    height="170"
  />
</a>
```

## Badge Contents

The badge is a 400x170px SVG that includes:

| Element | Description |
|---|---|
| QR code | Links to the widget URL, rendered as inline SVG rectangles |
| Widget name | The widget's display name (e.g. "Project Donations") |
| Stats | Total XMR received and transaction count (if `show_stats` is enabled) |
| Footer | "Powered by Tiponero" text |

## Theme

The badge respects the widget's theme setting:

| Theme | Badge style |
|---|---|
| `dark` | Dark background (`#201F1F`), light text |
| `light` | Light background, dark text |
| `system` | Uses CSS `prefers-color-scheme` media query |

## Dynamic Content

The badge is generated dynamically on each request. Stats (total received, transaction count) are always current. There is no caching: each request queries the database for the latest data.

If you need to display the badge in contexts where SVG rendering is limited (e.g., some email clients), consider taking a screenshot or converting to PNG.
