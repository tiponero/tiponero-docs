---
sidebar_position: 2
title: SVG Badge
---

# SVG Badge

Every widget has a live SVG badge endpoint. You can embed the badge in markdown documents, README files, or any website. The badge shows the widget name and a QR code that links to the widget. It can also show the total received amount and the transaction count.

## Badge URL

```
https://your-instance.example.com/widget/<widget-id>/badge.svg
```

The badge is public and needs no authentication.

## Embedding in Markdown

Link the badge image to the widget URL. A visitor can then click the badge and donate:

```markdown
[![Donate with Monero](https://your-instance.example.com/widget/f47ac10b-58cc-4372-a567-0e02b2c3d479/badge.svg)](https://your-instance.example.com/widget/f47ac10b-58cc-4372-a567-0e02b2c3d479)
```

The badge becomes clickable in GitHub READMEs, GitLab, and any markdown renderer that accepts external images.

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

The badge is an SVG image of 400x170px. It contains:

| Element | Description |
|---|---|
| QR code | Links to the widget URL, rendered as inline SVG rectangles |
| Widget name | The widget display name (for example, "Project Donations") |
| Stats | Total XMR received and transaction count (if `show_stats` is enabled) |
| Footer | "Powered by Tiponero" text |

## Theme

The badge uses the theme setting of the widget:

| Theme | Badge style |
|---|---|
| `dark` | Dark background (`#201F1F`), light text |
| `light` | Light background, dark text |
| `system` | Uses CSS `prefers-color-scheme` media query |

## Dynamic Content

Tiponero generates the badge on each request. The stats (total received, transaction count) are always current. Tiponero does not cache the badge: each request queries the database for the latest data.

Some contexts do not fully render SVG, for example some email clients. In those contexts, take a screenshot of the badge or convert it to PNG.
