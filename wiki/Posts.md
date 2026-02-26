---
title: Posts
---

Posts are blog-style content - articles, updates, announcements, recaps. Each post is a Markdown file in the `src/posts/` folder.

---

## Creating a Post

1. Copy `src/posts/template.md` and rename it (e.g. `src/posts/my-first-post.md`).
2. Edit the front matter and content:

```markdown
---
title: My First Post
description: A quick update on what we've been up to.
postDate: 2026-02-24
author: Jane Smith
---

Write your post content here using Markdown.

## A Subheading

More content, **bold text**, [links](https://example.com), images, whatever you need.
```

3. Save the file. The post will appear automatically on the posts listing page and on the homepage.

---

## Front Matter Properties

| Property | Required | Description |
|---|---|---|
| `title` | Yes | The post title. |
| `description` | Recommended | A short summary shown on post cards and in search results. It also appears in search result previews and in SEO meta tags. |
| `postDate` | Yes | The publication date in `YYYY-MM-DD` format (e.g. `2026-02-24`). Used for sorting - newest posts appear first. |
| `author` | No | The author name, shown on the post page. |
| `image` | No | Path to a header image shown on the post card (e.g. `/assets/images/my-photo.jpg`). |
| `preview_image` | No | A separate image used in search result previews. Falls back to `image` if not set. |
| `featured` | No | Set to `true` to mark as featured. Featured posts appear in the featured section (if enabled on the homepage). |
| `gallery` | No | Path to a folder of images to display as a gallery on the post. See [[Gallery]]. |
| `eventPage` | No | Link to a related event page. Displays a "Go to event page" link on the post. |

> **Title and description matter.** The title is what appears in browser tabs, search results, and when someone shares your post. The description appears in search result previews and SEO meta tags. Writing a clear, specific title and description for every post makes your site much more useful and findable. See [[Search]] for how the search system uses these fields.

---

## How Posts Are Displayed

### Posts Listing Page

The `src/posts/posts.njk` file generates the `/posts/` page, which shows all posts sorted by date (newest first). You generally don't need to edit this file.

### Homepage Preview

The homepage automatically shows the 3 most recent posts via the `homeCards.njk` partial. This is controlled by the `previewPostsAll` collection in `.eleventy.js`.

### Individual Post Pages

Each post gets its own page at `/posts/your-filename/`. The URL is derived from the filename - `my-first-post.md` becomes `/posts/my-first-post/`.

Posts use the `post.njk` layout, which is set automatically by the `posts.json` directory data file. You don't need to specify a layout in each post.

---

## Featured Posts

To feature a post, add `featured: true` to its front matter:

```yaml
---
title: Big Announcement
postDate: 2026-02-24
featured: true
---
```

Featured posts can be displayed separately on the homepage using the featured partial. By default, the homepage shows the latest posts regardless of featured status - the featured section is available but commented out in `homeCards.njk`. You can enable it by uncommenting it.

---

## Linking Posts to Events

If a post is a recap of an event, you can link them together:

**In the post**, add `eventPage` pointing to the event:

```yaml
---
title: Game Night Recap
postDate: 2026-02-25
eventPage: /events/game-night/
---
```

**In the event**, add `recapPost` pointing to the post:

```yaml
---
title: Game Night
eventDate: 2026-02-24
recapPost: /posts/game-night-recap/
---
```

Both pages will show a link to the other.

---

## Tips

- **Hiding the template file** - Rename `src/posts/template.md` to `template.njk` to hide it from the posts listing. The posts collection only scans `*.md` files, so `template.njk` won't appear in your posts list.
- **Filenames matter** - The filename becomes the URL. Use lowercase with hyphens: `my-post-title.md` → `/posts/my-post-title/`.
- **Date format** - Always use `YYYY-MM-DD` for `postDate`. The display format is controlled by `site.json`'s `dateFormat` setting.
- **Images** - Place post images in `src/assets/images/` and reference them as `/assets/images/filename.jpg`.
- **Markdown is flexible** - You can use HTML within Markdown files when needed, including Bootstrap classes.
