---
title: Customisation
---

SpaceClub! is designed to be rebranded from a handful of files. You don't need to dig into HTML templates - most visual changes come from editing `site.json`, `navbar.json`, and the top of `styles.css`.

---

## Site Identity

Open `src/_data/site.json`:

```json
{
  "title": "Space Club!",
  "description": "Your site description here.",
  "logo": "/assets/images/logo.svg",
  "favicon": "/assets/images/favicon.svg",
  "baseUrl": "/",
  "timezone": "Australia/Sydney",
  "dateFormat": "dddd, Do MMMM YYYY",
  "searchTitle": "Search",
  "searchPlaceholder": "Search content..."
}
```

| Property | What It Controls |
|---|---|
| `title` | The site name shown in the navbar, footer, and browser tab. |
| `description` | The default meta description for SEO. Individual pages can override this. |
| `logo` | Path to your logo image. Displayed in the navbar. |
| `favicon` | Path to the favicon (the small icon in browser tabs). |
| `baseUrl` | The base URL path. Leave as `"/"` unless hosting in a subdirectory. |
| `timezone` | Used for event date calculations (upcoming vs. past). Use a [TZ database name](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones). |
| `dateFormat` | The default format for displayed dates. Uses [Day.js format tokens](https://day.js.org/docs/en/display/format) - e.g. `"MMMM D, YYYY"` for "January 5, 2026". |
| `searchTitle` | The heading shown in the search modal. |
| `searchPlaceholder` | The placeholder text in the search input field. |

---

## Navigation

Open `src/_data/navbar.json`:

```json
[
  {
    "title": "Home",
    "href": "/",
    "footer": true
  },
  {
    "title": "About",
    "href": "/about",
    "footer": true
  },
  {
    "title": "Contact",
    "href": "/contact",
    "footer": true
  }
]
```

Each item is a navigation link:

- `title` - The text shown in the navbar and footer.
- `href` - The URL path the link points to.
- `footer` - Set to `true` to also show this link in the footer. Set to `false` or remove it to only show it in the navbar.

To add a new link, add a new object to the array. For example, to add an Events link:

```json
{
  "title": "Events",
  "href": "/events",
  "footer": false
}
```

The navbar automatically highlights the current page.

---

## Colours

Open `src/assets/css/styles.css`. The colour scheme is controlled by CSS custom properties at the very top:

```css
:root {
  --bs-primary: #d4204d;
  --bs-primary-rgb: 212, 32, 77;
  --bs-primary-text-emphasis: #a00c31;
  --bs-secondary: #8200d8;
  --bs-secondary-rgb: 130, 0, 216;
  --bs-secondary-text-emphasis: #4a0f72;
  --bs-heading-color: var(--bs-primary);
}
```

To change your colour scheme, update these values:

| Variable | What It Affects |
|---|---|
| `--bs-primary` | The main brand colour - buttons, links, headings, navbar highlights. |
| `--bs-primary-rgb` | The same colour as RGB values (needed for Bootstrap opacity utilities). |
| `--bs-primary-text-emphasis` | A darker shade used for hover states. |
| `--bs-secondary` | The accent colour - used for featured items and secondary buttons. |
| `--bs-secondary-rgb` | The same colour as RGB values. |
| `--bs-secondary-text-emphasis` | A darker shade for hover states. |
| `--bs-heading-color` | The colour of all headings. Defaults to primary. |

**Tip:** Use a tool like [coolors.co](https://coolors.co) to generate a colour palette, then convert the hex values to RGB for the `-rgb` variants.

### The Footer Gradient

The footer has a gradient overlay defined in this same file:

```css
--gradient: linear-gradient(135deg, #3BC7F6 0%, #A855F7 29.8%, #CC4D9E 69.3%, #EF4444 100%);
```

Change these colour stops to match your brand, or simplify it to a solid colour:

```css
--gradient: linear-gradient(135deg, var(--bs-primary) 0%, var(--bs-secondary) 100%);
```

---

## Fonts

The template uses a custom font loaded from `src/assets/fonts/`. To change it:

1. Download your font as a `.ttf`, `.woff`, or `.woff2` file.
2. Place it in `src/assets/fonts/`.
3. Update the `@font-face` block in `styles.css`:

```css
@font-face {
  font-family: "YourFont";
  src: url("/assets/fonts/YourFont.woff2") format("woff2");
  font-weight: 100 900;
  font-style: normal;
}

body {
  font-family: "YourFont", var(--bs-body-font-family);
}
```

The `var(--bs-body-font-family)` fallback ensures the site still looks fine if the custom font fails to load.

**Where to find fonts:** [Google Fonts](https://fonts.google.com) is free. Download the font files directly rather than using the `<link>` embed - this way your site works offline and doesn't depend on Google's servers.

---

## Logo and Favicon

Replace these files in `src/assets/images/`:

- `logo.svg` - Your site logo, displayed in the navbar (rendered at 132×56px).
- `favicon.svg` - The small icon in browser tabs.

SVG is recommended because it scales to any size. If you use PNG or JPG, make sure the logo is at least 264×112px (2× for retina displays) and the favicon is at least 32×32px.

---

## Footer Divider

The footer uses a custom SVG divider defined in the CSS:

```css
--divider-url: url('/assets/space-divider.svg');
```

To change or remove it:

- **Replace it** - Create your own SVG divider and update the path.
- **Remove it** - Delete the `--divider-url` line and the `.divider` and `.divider::before`/`::after` CSS rules, or simply don't include the divider partial in your footer.

---

## Homepage Hero Text

The homepage heading and intro text are styled with specific sizes in the CSS:

```css
.homepage-hero h1:first-child {
  font-size: 3.5rem;
  font-weight: 300;
  line-height: 1.2;
}
```

You can adjust these values to change the feel of your landing page. The content itself is in `src/index.md`.

---

## Going Further

For advanced customisation like building your own card list components from a JSON data file or from a folder of Markdown files, see **[[Advanced Customisation]]**.
