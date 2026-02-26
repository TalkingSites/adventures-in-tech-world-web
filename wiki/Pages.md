---
title: Pages
---

Pages are the standalone content of your site - things like your homepage, about page, and contact page. Each page is a Markdown file in the `src/` folder.

---

## How Pages Work

Every page is a `.md` (Markdown) file with two parts:

1. **Front matter** - Settings at the top between `---` lines.
2. **Content** - Your page content below the front matter, written in Markdown.

Here's the about page as an example:

```markdown
---
layout: base.njk
title: About Us
description: Learn more about what we do.
---

# About Us

This is our story...
```

### Front Matter Properties

| Property | Required | Description |
|---|---|---|
| `layout` | Yes | The layout template to use. Use `base.njk` for standard pages. |
| `title` | Yes | The page title - shown in the browser tab and in search results. |
| `description` | No | A short description for SEO meta tags and search results. |

The URL for the page is automatically derived from the filename — `src/faq.md` becomes `/faq/`. You don't need to set it manually.

---

## Existing Pages

The template comes with these pages:

| File | URL | Purpose |
|---|---|---|
| `src/index.md` | `/` | Homepage |
| `src/about.md` | `/about/` | About page |
| `src/contact.md` | `/contact/` | Contact page (includes a form) |
| `src/404.md` | `/404.html` | Page not found |

---

## Creating a New Page

1. Create a new `.md` file in `src/`. For example, `src/faq.md`.
2. Add front matter at the top:

```markdown
---
layout: base.njk
title: FAQ
description: Frequently asked questions.
---

# Frequently Asked Questions

## How do I change the site name?

Edit `src/_data/site.json` and change the `title` field.
```

3. Save the file. If the dev server is running (`npm start`), the page will be available immediately at `/faq/`.
4. To add it to the navigation, add an entry to `src/_data/navbar.json`:

```json
{
  "title": "FAQ",
  "href": "/faq",
  "footer": false
}
```

---

## The Homepage

The homepage (`src/index.md`) is slightly different from other pages:

- It uses the `index.njk` layout (which wraps `base.njk` but adds a hero section).
- It includes the `homeCards.njk` partial, which automatically shows your latest posts and upcoming events.

```markdown
---
layout: index.njk
title: Welcome!
description: Check out our homepage
---

# Hello World!

Your intro text here.

<div>
{% include "partials/homeCards.njk" %}
</div>
```

The `homeCards.njk` partial pulls in the 3 most recent posts and upcoming events automatically. You don't need to manage those lists manually.

---

## The Contact Page

The contact page is a regular page that includes the form partial. See [[Forms]] for full details on how to configure form fields.

---

## Tips

- **Custom URL** - If you want a page to be at a different URL than its filename, add `permalink: "/custom-path/"` to the front matter (always include trailing slashes). The 404 page uses this — its permalink is set to `/404.html` because that's the filename Netlify expects.
- **Using HTML in Markdown** - You can mix HTML into your Markdown content when you need more control. For example, you can use Bootstrap classes directly: `<div class="alert alert-info">Note!</div>`.
- **Including partials** - You can include any Nunjucks partial in a Markdown file using `{% include "partials/filename.njk" %}`. This is how the contact page includes the form.
- **`.md` vs `.njk` files** - Pages can be either `.md` or `.njk`. Start with `.md` since it's cleaner for content-focused pages. If you find yourself adding a lot of HTML and Nunjucks template code to a page, consider renaming it to `.njk`. Both work identically, but `.njk` signals to anyone reading the project that the file is more of a template than a content document. It's a small thing, but it makes the project easier to navigate as it grows.

## The `homeCards.njk` Partial

The homepage includes `homeCards.njk`, which renders the latest posts and upcoming events on the home page. This is just a convenience partial — it's entirely yours to change.

Open `src/_includes/partials/homeCards.njk` and customise it however you like:
- Remove the posts section if you don't have a blog.
- Remove the events section if you don't run events.
- Add any other partial you want — gallery, featured items, a custom list.
- Or remove the `{% include "partials/homeCards.njk" %}` line from `src/index.md` entirely if you'd rather build the homepage layout yourself from scratch.

See [[Advanced Customisation]] for how to build your own card list components.
