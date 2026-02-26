---
title: Buttons
---

The `btn.njk` partial creates a styled link button using Bootstrap. You can use it anywhere in a page, post, or event - and you can include it multiple times on one page with different settings each time.

---

## Basic Usage

Set the variables you want to customise immediately before the include. Any variable you don't set will use its default.

```nunjucks
{% set text = "Register Now" %}
{% set link = "https://example.com/register" %}
{% set icon = "ticket-perforated" %}
{% set style = "primary" %}
{% include "partials/btn.njk" %}
```

---

## Options

| Variable | Default | Description |
|---|---|---|
| `text` | `"Back to Home"` | The button label. |
| `link` | `"/"` | The URL the button links to. Can be a relative path (`/events/`) or a full URL (`https://...`). |
| `icon` | `"arrow-left"` | A [Bootstrap Icon](https://icons.getbootstrap.com/) name, without the `bi-` prefix. Set to `""` or `false` to show no icon. |
| `style` | `"primary"` | A Bootstrap button style. Common values: `primary`, `outline-primary`, `secondary`, `outline-secondary`, `danger`, `outline-danger`. See the [Bootstrap buttons docs](https://getbootstrap.com/docs/5.3/components/buttons/) for all options. |
| `align` | `"center"` | Horizontal alignment of the button: `start`, `center`, or `end`. |

---

## Multiple Buttons on One Page

Set variables fresh before each include. Each `{% include %}` is independent:

```nunjucks
{% set text = "Register Now" %}
{% set link = "https://example.com/register" %}
{% set icon = "ticket-perforated" %}
{% set style = "primary" %}
{% include "partials/btn.njk" %}

{% set text = "View on Map" %}
{% set link = "https://maps.google.com/?q=your+address" %}
{% set icon = "geo-alt" %}
{% set style = "outline-primary" %}
{% include "partials/btn.njk" %}

{% set text = "Contact Us" %}
{% set link = "/contact/" %}
{% set icon = "envelope" %}
{% set style = "outline-secondary" %}
{% include "partials/btn.njk" %}
```

---

## Default Button on Event and Post Pages

The event layout (`src/_includes/layouts/event.njk`) and post layout (`src/_includes/layouts/post.njk`) both automatically include `btn.njk` at the bottom of every page with no variables set, so it falls back to its defaults: a "Back to Home" arrow button linking to `/`.

To change the default for all events, edit the layout file directly. To add extra buttons on a specific page, include the partial in your page's Markdown content with `{% set %}` variables as shown above - these appear in the content area, separate from the default button at the bottom.

---

## Finding Icons

Browse all available icons at [icons.getbootstrap.com](https://icons.getbootstrap.com/). Use the icon name without the `bi-` prefix. For example, the icon `bi-arrow-right` becomes `icon = "arrow-right"`.
