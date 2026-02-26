---
title: Hosting and Deployment
---

Once you've built your site, deploying it means uploading the `_site/` folder to a hosting service. Since SpaceClub! is a static site, you have many options - most of them free.

---

## Building Your Site

Before deploying, run the build command:

```bash
npm run build
```

This does two things:
1. Runs Eleventy to generate your site in `_site/`.
2. Runs Pagefind to create the search index.

The `_site/` folder contains your complete website - HTML, CSS, JavaScript, images, everything. You can upload this folder to any static hosting service.

---

## Netlify (Recommended)

Netlify is the easiest option. It integrates with GitHub, automatically builds and deploys your site on every push, and includes form handling.

### Deploy from GitHub

1. Push your project to a GitHub repository.
2. Go to [netlify.com](https://www.netlify.com) and sign up (free).
3. Click **"Add new site" → "Import an existing project"**.
4. Select your GitHub repository.
5. Netlify will auto-detect the settings from `netlify.toml`:
   - **Build command:** `npx @11ty/eleventy && npx pagefind --site _site`
   - **Publish directory:** `_site`
6. Click **"Deploy site"**.

That's it. Every time you push to your repository, Netlify will automatically rebuild and deploy.

### Custom Domain

In Netlify: **Site settings → Domain management → Add custom domain**. Netlify provides free HTTPS via Let's Encrypt.

### Forms

The SpaceClub! contact form includes the `netlify` attribute, so form submissions are automatically handled by Netlify. View submissions in your Netlify dashboard under **Forms**.

### The `netlify.toml` File

Your project already includes this file with the correct settings:

```toml
[build]
  publish = "_site"
  command = "npx @11ty/eleventy && npx pagefind --site _site"

[build.environment]
  NODE_VERSION = "20"
```

---

## Other Hosting Options

Since SpaceClub! outputs a plain folder of static files, it works with any static hosting service. Some popular options:

- **[GitHub Pages](https://pages.github.com/)** - Free, integrates with GitHub. No built-in form handling (see [[Forms]] for alternatives).
- **[Vercel](https://vercel.com)** - Free tier, GitHub integration, fast CDN. No built-in form handling.
- **[Cloudflare Pages](https://pages.cloudflare.com/)** - Free, fast global CDN.
- **Manual / FTP** - Build locally with `npm run build`, then upload the `_site/` folder to any web host.

For all of these, the build command is `npm run build` and the output directory is `_site/`. These options have not been verified with SpaceClub! specifically, so you may need to adjust configuration.

For a full list of Eleventy-compatible deployment options: [11ty.dev/docs/deployment](https://www.11ty.dev/docs/deployment/)

---

## Drag-and-Drop Deploy

Netlify also supports drag-and-drop: go to [app.netlify.com/drop](https://app.netlify.com/drop) and drag your `_site/` folder onto the page. Your site will be live in seconds. Great for quick tests.

---

## Summary

| Host | Free | Auto-Deploy | Forms | Verified with SpaceClub! |
|---|---|---|---|---|
| **Netlify** | ✅ | ✅ | ✅ Built-in | ✅ |
| **GitHub Pages** | ✅ | ✅ (via Actions) | ❌ | Not verified |
| **Vercel** | ✅ | ✅ | ❌ | Not verified |
| **Cloudflare Pages** | ✅ | ✅ | ❌ | Not verified |
| **Manual/FTP** | Depends | ❌ | ❌ | Not verified |
