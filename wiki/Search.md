---
title: Search
---

SpaceClub! includes full-text search powered by [Pagefind](https://pagefind.app/). Users click the Search button in the navbar, type a query, and get instant results - no server required.

---

## How It Works

1. **At build time** - When you run `npm run build`, Pagefind scans the `_site/` output and creates a search index.
2. **At runtime** - When a user types in the search modal, the `search.js` script loads the Pagefind index and performs client-side search.

Everything happens in the browser. There's no server, no API, no third-party service.

---

## Using Search

Users click the **Search** button in the navbar (or you can trigger it with `openSearch()` from JavaScript). This opens a modal with a search input. As they type, results appear below.

Each result shows:
- The page title.
- A description or text excerpt.
- Tags (if any).
- A preview image (if the page has a `preview_image` property).

Clicking a result navigates to that page.

---

## What Gets Indexed

Pagefind indexes everything inside the `data-pagefind-body` container in the base layout. This includes:
- Page titles and descriptions (from front matter).
- All page content (the Markdown you write).

The title and description are also set as `data-pagefind-meta` attributes in the `<head>`, so they appear as structured metadata in search results.

---

## Title-First Ranking

The search uses a custom ranking system that prioritises title matches. This means if you search for "game night," a page titled "Game Night" will rank above a page that merely mentions "game night" in its body text.

The ranking works by:
1. Checking for exact title matches (highest boost).
2. Checking if the title starts with the search query.
3. Checking if all search terms appear in the title.
4. Checking if any search terms appear in the title.
5. Gentle boosts for description and body text matches.
6. Tie-breaking by shorter titles and original Pagefind order.

---

## Customising the Search Modal

The search modal title and placeholder text are configurable in `src/_data/site.json`:

```json
{
  "searchTitle": "Search",
  "searchPlaceholder": "Search content..."
}
```

Change these to match your site - for example, a café site might use:

```json
{
  "searchTitle": "Find Something",
  "searchPlaceholder": "Search our menu, events, and blog..."
}
```

---

## Important: Search Only Works After a Build

During development (`npm start`), the Pagefind index doesn't exist, so the search modal will show an error or no results. This is normal.

To test search locally:

```bash
npm run build
npx serve _site
```

This builds the site (including the Pagefind index) and serves it locally. Search will work on the served version.

When your site is deployed (e.g. to Netlify), the build command runs automatically and search works out of the box.

---

## How Tags Appear in Search

If your posts or events have tags (either in front matter metadata or Eleventy filters), Pagefind picks them up and the search results display them as badges. Generic tags like "post" or "page" are automatically filtered out to keep results clean.

---

## Tips

- **Write good titles and descriptions** - These are what appear in search results and what the ranking algorithm prioritises. Clear, specific titles make search much more useful.
- **Preview images** - Add `preview_image` to a post or event's front matter to show a thumbnail in search results.
- **Pagefind documentation** - For advanced configuration (like excluding certain pages from search), see the [Pagefind documentation](https://pagefind.app/).
