---
title: Quick Start
---

Already familiar with Git, Node.js, and the command line? Here's the fastest path.

## 1. Clone and Run

```bash
git clone https://github.com/ArcadeFortune/spaceclub-template.git
cd spaceclub-template
npm install
npm start
```

Your site is now running at `http://localhost:8080`. Changes you make to files in `src/` will auto-reload in the browser.

## 2. Make It Yours

Open these files and change them to match your site:

| File | What to Change |
|---|---|
| `src/_data/site.json` | Site title, description, logo, favicon, timezone, date format |
| `src/_data/navbar.json` | Navigation links (and which ones appear in the footer) |
| `src/assets/css/styles.css` | Primary and secondary colours (the CSS variables at the top) |
| `src/assets/fonts/` | Replace the font file and update the `@font-face` in `styles.css` |
| `src/assets/images/` | Replace `logo.svg` and `favicon.svg` with your own |

## 3. Edit Content

- **Homepage** - Edit `src/index.md`
- **About page** - Edit `src/about.md`
- **Contact page** - Edit `src/contact.md`
- **Add a post** - Copy `src/posts/template.md`, rename it, and edit the front matter and content
- **Add an event** - Copy `src/events/template.md`, rename it, and edit the front matter and content

## 4. Build for Production

```bash
npm run build
```

This runs Eleventy to generate your site, then runs Pagefind to index it for search. The output goes to `_site/` - that folder is your complete website, ready to upload anywhere.

## 5. Deploy

The template includes a `netlify.toml` for one-click Netlify deployment. See [[Hosting and Deployment]] for all options including GitHub Pages and Vercel.

## Next Steps

- **[[Customisation]]** - Deep dive into colours, fonts, and branding
- **[[Events]]** - Full event system documentation
- **[[Forms]]** - How the contact form system works
- **[[Gallery]]** - Adding image galleries to posts and events
