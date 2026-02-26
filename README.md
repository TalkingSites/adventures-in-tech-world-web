# Adventures in Tech World — Website

Website for **Adventures in Tech World**, an educational multiplayer 2D pixel game where players explore, solve coding challenges, and collaborate in real time. Play free in the browser.

Based on the [SpaceClub template](https://github.com/TalkingSites/spaceclub-template) by TalkingSites.
Built with [Eleventy](https://www.11ty.dev/), Markdown, Bootstrap 5, and Pagefind. 

**Game repository:** [github.com/enspyr/tech-world](https://github.com/enspyr/tech-world)

---

## Development

[SpaceClub wiki](spaceclubwiki.talkingsites.org)

Install dependencies:

```bash
npm install
```

Start the local dev server:

```bash
npm start
```

Build for production:

```bash
npm run build
```

## Updating from the template

This repo tracks the SpaceClub template as an `upstream` remote (fetch only — pushes are blocked):

```bash
git fetch upstream
git merge upstream/main
```

## Deployment

Deployed via [Netlify](https://netlify.com). See `netlify.toml` for configuration.

Netlify Adventure In Tech World website: [https://aitw.netlify.app](https://aitw.netlify.app)