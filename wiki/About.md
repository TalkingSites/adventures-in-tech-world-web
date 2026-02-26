---
title: About SpaceClub!
---

## What Is SpaceClub!?

SpaceClub! is a free, open-source website template. You use it to create a website that you **fully own** - the files live on your computer (or in your GitHub account), you can edit everything, and you can host it wherever you like.

There's no account to sign up for, no monthly fee, and no platform that can change the rules on you.

## Who Is It For?

SpaceClub! is designed for two types of people:

1. **Developers** who want a clean, well-structured Eleventy starter with Bootstrap, search, events, posts, galleries, and forms already wired up - so they can skip the boilerplate and start customising immediately.

2. **Non-developers** who've been burned by website builders like Squarespace or Wix - unexpected price increases, features removed overnight, limited control - and now want to own their website for real. If that's you: yes, there's a small learning curve, but it's very doable. The [[Walkthrough]] will guide you through every step, and AI tools like ChatGPT or GitHub Copilot can help you along the way.

## What's It Built With?

You don't need to deeply understand these to use SpaceClub!, but here's what's under the hood:

| Technology | What It Does |
|---|---|
| **[Eleventy](https://www.11ty.dev/)** | The engine that turns your content files into a website. It's a static site generator - you write simple text files (Markdown), and Eleventy builds them into HTML pages. |
| **[Markdown](https://www.markdownguide.org/)** | The format you write your content in. It's plain text with simple formatting like `# Heading` and `**bold**`. No HTML required for everyday editing. |
| **[Nunjucks](https://mozilla.github.io/nunjucks/)** | The templating language used for layouts and reusable components. You won't need to write Nunjucks - it's already set up - but you'll see it in layout files. |
| **[Bootstrap 5](https://getbootstrap.com/)** | The CSS framework that makes the site look good and work on all screen sizes. SpaceClub! uses Bootstrap's utility classes and components throughout. |
| **[Pagefind](https://pagefind.app/)** | Adds search to your site. It indexes your content at build time and provides fast, client-side search with no server required. |

## What Is a Static Site?

A static site is a website made of plain HTML, CSS, and JavaScript files - no database, no server-side code, no WordPress. You write your content, run a build command, and get a folder of files that *are* your website.

**Why this matters:**

- **It's fast** - there's no server processing each page load, just files being served directly.
- **It's secure** - there's no database to hack, no admin panel to break into.
- **It's portable** - the output is just files. You can host them literally anywhere: Netlify, GitHub Pages, Vercel, a USB stick, your own server.
- **It's yours** - if your hosting provider shuts down tomorrow, you still have all your files. Upload them somewhere else and you're back online.

The trade-off is that you don't have a drag-and-drop visual editor. Instead, you edit text files. But those text files are Markdown, which is designed to be readable and simple - and once you get the hang of it, it's faster than any visual builder.

## How SpaceClub! Is Structured

Here's what the project looks like:

```
src/
├── index.md              ← Your homepage
├── about.md              ← About page
├── contact.md            ← Contact page
├── 404.md                ← Page not found
├── _data/
│   ├── site.json         ← Site title, description, timezone
│   ├── navbar.json       ← Navigation links
│   └── global.js         ← Global data (e.g. current year)
├── _includes/
│   ├── layouts/          ← Page layouts (base, post, event)
│   └── partials/         ← Reusable components (navbar, footer, forms, gallery)
├── assets/
│   ├── css/styles.css    ← Your styles and colour customisation
│   ├── fonts/            ← Custom fonts
│   ├── images/           ← Images and logos
│   └── js/search.js      ← Search functionality
├── events/               ← Event pages (one .md file per event)
│   ├── events.json       ← Default layout for events
│   ├── events.njk        ← Events listing page
│   └── template.md       ← Starter template for new events
└── posts/                ← Blog posts (one .md file per post)
    ├── posts.json        ← Default layout for posts
    ├── posts.njk         ← Posts listing page
    └── template.md       ← Starter template for new posts
```

**The key idea:** you mostly work in `src/`. You edit `.md` files to change content, `.json` files to change configuration, and `.css` to change appearance. The `_includes/` folder has the layout and component templates, but you rarely need to touch those unless you're customising the design itself.

## Features at a Glance

- ✅ **Pages** - Create any page with Markdown. See [[Pages]].
- ✅ **Blog Posts** - Write posts with dates, authors, and images. See [[Posts]].
- ✅ **Events** - Full event system with dates, times, locations, upcoming/past sorting. See [[Events]].
- ✅ **Contact Forms** - Data-driven forms with validation and URL prefilling. See [[Forms]].
- ✅ **Image Galleries** - Per-page galleries with lightbox viewer. See [[Gallery]].
- ✅ **Search** - Full-text search powered by Pagefind. See [[Search]].
- ✅ **Customisation** - Colours, fonts, navigation, and branding from a few config files. See [[Customisation]].
- ✅ **Hosting** - Deploy anywhere: Netlify, GitHub Pages, Vercel, and more. See [[Hosting and Deployment]].
