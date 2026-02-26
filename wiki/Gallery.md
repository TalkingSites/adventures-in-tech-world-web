---
title: Gallery
---

SpaceClub! has a built-in image gallery system. You can attach a gallery to any event or post by pointing it at a folder of images. The template handles the grid layout, thumbnails, and a lightbox viewer with keyboard navigation.

---

## Adding a Gallery to a Post or Event

1. Create a folder for your images inside `src/assets/images/` (e.g. `src/assets/images/open-day/`).
2. Add your images to that folder (`.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`, or `.svg`).
3. Set the `gallery` property in your post or event front matter:

```yaml
---
title: Open Day Recap
postDate: 2026-05-16
gallery: "/assets/images/open-day"
---
```

That's it. The gallery will appear at the bottom of the post or event page.

---

## How It Works

When a post or event has a `gallery` property, the layout template automatically includes the `gallery.njk` partial. This partial:

1. Scans the specified folder for image files.
2. Renders a responsive grid of thumbnails.
3. Generates a lightbox modal for viewing full-size images with previous/next navigation and keyboard arrow key support.

Images are sorted alphabetically by filename. If you want a specific order, prefix your filenames with numbers: `01-first.jpg`, `02-second.jpg`, etc.

---

## Gallery in the Page Content

If you want the gallery to appear at a specific position within your content (rather than at the bottom), you can include it manually:

```markdown
---
title: Our Trip
postDate: 2026-05-16
gallery: "/assets/images/trip"
---

# Our Trip

Here are some photos from the day:

{% set path = "assets/images/trip" %}
{% include "partials/gallery.njk" %}

And here's what happened next...
```

When you manually include the gallery partial, the automatic gallery at the bottom is suppressed (it detects that a gallery is already present in the content).

**Note:** When using `{% set path %}`, omit the leading slash - use `assets/images/trip` not `/assets/images/trip`.

---

## Gallery with a Filter Prefix

If a folder contains many images and you only want to show some of them, you can filter by filename prefix:

```nunjucks
{% set path = "assets/images/mixed-folder" %}
{% set filterPrefix = "gallery-" %}
{% include "partials/gallery.njk" %}
```

This will only show images whose filenames start with `gallery-`.

---

## The All-Images Gallery

SpaceClub! also includes an aggregated gallery (`galleryAll.njk`) that pulls images from *all* posts and events that have a `gallery` property. This is useful for a dedicated photos page.

To use it, create a gallery page:

```markdown
---
layout: base.njk
title: Photo Gallery
permalink: "/gallery/"
---

# All Photos

{% include "partials/galleryAll.njk" %}
```

### Limiting the Preview

You can show a limited number of images with a "View All" link:

```nunjucks
{% set galleryLimit = 12 %}
{% include "partials/galleryAll.njk" %}
```

This shows 12 images with a link to the full gallery page.

The all-images gallery lightbox also shows which post or event each image belongs to, with a link to that page.

---

## Supported Image Formats

The gallery system supports these file types:

- `.jpg` / `.jpeg`
- `.png`
- `.gif`
- `.webp`
- `.svg`

---

## Tips

- **Image size** - Thumbnails are displayed at 120px tall with `object-fit: cover`. For best results, use images that are at least 400px wide.
- **Organisation** - Create a subfolder per event or post in `src/assets/images/`. This keeps things tidy and makes the `gallery` property intuitive.
- **Filenames** - Avoid spaces in image filenames. Use hyphens or underscores: `open-day-01.jpg`.
- **Alt text** - The gallery generates alt text automatically from the filename (replacing hyphens and underscores with spaces). For better accessibility, use descriptive filenames.
