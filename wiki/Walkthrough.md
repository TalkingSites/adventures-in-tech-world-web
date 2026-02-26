---
title: Walkthrough
---

This guide is for you if you're new to building websites this way. Maybe you've used Squarespace or Wix before, or maybe you've never made a website at all. Either way, we'll walk through everything step by step.

> **Don't be intimidated.** SpaceClub! was specifically designed so you only need to edit simple text files. And if you get stuck, AI tools like ChatGPT or GitHub Copilot can explain anything you see - just paste the file and ask.

---

## Step 1: Understand What We're Building

SpaceClub! is a **static site**. That means:

- Your website is a collection of files (text, images, CSS) that live in a folder.
- You edit those files to change your website.
- You run a command to "build" those files into a finished website.
- You upload the finished website to a hosting service - and that's it, you're live.

There's no database. There's no login screen for your website. There's no admin panel. Just files.

**Why is this better?**

- **You own everything.** The files are yours. No company can take them away or change how they work.
- **It's free to host.** Services like Netlify and GitHub Pages host static sites for free.
- **It's fast and secure.** No server code means nothing to hack and nothing to slow down.
- **It's portable.** Don't like your host? Move your files somewhere else. Done.

The trade-off is that you can't click and drag to rearrange things. You edit text files instead. But those text files use **Markdown**, which was designed to be simple and readable - and once you try it, you'll find it's actually faster than any visual editor.

---

## Step 2: What You Need

You'll need three things:

### A GitHub Account (Free)

GitHub is where the SpaceClub! template lives, and it's also a great place to store your website's code.

1. Go to [github.com](https://github.com) and sign up for a free account.
2. That's it for now.

### A Code Editor

You need something to edit the files. You have two options:

**Option A: GitHub Codespaces (easiest - works in your browser)**

GitHub Codespaces gives you a full code editor in your web browser. No installation needed.

1. Go to the [SpaceClub! template repository](https://github.com/ArcadeFortune/spaceclub-template).
2. Click the green **"Code"** button.
3. Click the **"Codespaces"** tab.
4. Click **"Create codespace on main"**.
5. Wait a minute - a full editor opens in your browser. You can skip to [[#Step 4 Install the Dependencies|Step 4]].

**Option B: VS Code (recommended for long-term use)**

VS Code is a free code editor that runs on your computer. Most developers use it.

1. Download VS Code from [code.visualstudio.com](https://code.visualstudio.com).
2. Install it like any other app.

### Node.js

Node.js is what runs the build tools behind the scenes. You don't need to learn Node.js - you just need it installed.

1. Go to [nodejs.org](https://nodejs.org).
2. Download the **LTS** (Long Term Support) version.
3. Install it like any other app.
4. To verify it worked, open a terminal and type:
   ```bash
   node --version
   ```
   If you see a version number (like `v20.11.0`), you're good.

> **What's a terminal?** It's a text-based way to talk to your computer. On Linux and Mac, it's called Terminal. On Windows, you can use Command Prompt or PowerShell. In VS Code, you can open one from the menu: **Terminal → New Terminal**.

---

## Step 3: Get the Template

Now you need to get the SpaceClub! files onto your computer (or into your Codespace).

### Option A: Use the Template Button (Recommended)

1. Go to the [SpaceClub! template repository](https://github.com/ArcadeFortune/spaceclub-template).
2. Click the green **"Use this template"** button near the top.
3. Click **"Create a new repository"**.
4. Give it a name (like `my-website`).
5. Click **"Create repository"**.
6. You now have your own copy on GitHub.

To get it onto your computer, open your terminal and run:

```bash
git clone https://github.com/YOUR-USERNAME/my-website.git
cd my-website
```

Replace `YOUR-USERNAME` with your actual GitHub username.

### Option B: Download as ZIP

If you don't want to use Git at all:

1. Go to the [SpaceClub! template repository](https://github.com/ArcadeFortune/spaceclub-template).
2. Click the green **"Code"** button.
3. Click **"Download ZIP"**.
4. Extract the ZIP file to a folder on your computer.
5. Open that folder in VS Code (**File → Open Folder**).

> **What is Git?** Git is a tool that tracks changes to your files over time. GitHub is a website that stores Git projects online. You don't need to be a Git expert - for now, just know it's how you get and save your files.

---

## Step 4: Install the Dependencies

SpaceClub! uses a few tools (like Eleventy and Pagefind) that need to be downloaded first. This is a one-time step.

Open a terminal in your project folder and run:

```bash
npm install
```

This reads the `package.json` file and downloads everything listed there. It'll create a `node_modules` folder - you can ignore that folder, it's just where the tools live.

> **What is npm?** npm stands for "Node Package Manager." It comes with Node.js and is used to install tools and libraries. The `npm install` command looks at `package.json` and downloads everything your project needs.

---

## Step 5: Start the Development Server

Now let's see your website! Run:

```bash
npm start
```

You'll see some output, and then a message like:

```
[11ty] Server at http://localhost:8080/
```

Open that URL in your browser. You should see the SpaceClub! template running locally on your computer.

**The best part:** leave this running. Every time you save a file, the browser will automatically update. This is called "live reload" and it makes editing much faster.

---

## Step 6: Edit Your First File

Let's make a change so you can see how it works.

1. Open `src/_data/site.json` in your editor.
2. Change the `title` from `"Space Club!"` to your website's name.
3. Save the file.
4. Look at your browser - it should have updated automatically.

That's the core workflow: **edit a file → save → see the result**.

Now try editing `src/index.md` - that's your homepage. Change the heading, change the text, save, and watch it update.

---

## Step 7: Understand the File Structure

Here's what matters:

| Folder/File | What It Is |
|---|---|
| `src/index.md` | Your homepage |
| `src/about.md` | Your about page |
| `src/contact.md` | Your contact page |
| `src/_data/site.json` | Site title, description, logo, timezone |
| `src/_data/navbar.json` | Navigation menu links |
| `src/assets/css/styles.css` | Colours and visual styles |
| `src/assets/images/` | Your logo, favicon, and images |
| `src/events/` | Event pages (one file per event) |
| `src/posts/` | Blog posts (one file per post) |

Everything you'll regularly edit is simple text. The `_includes/` folder has the HTML templates that control the layout - you generally don't need to touch those.

---

## Step 8: Understand Front Matter

At the top of every `.md` file, you'll see something like this:

```yaml
---
title: About Us
description: Learn more about what we do.
permalink: "/about/"
---
```

This is called **front matter**. It's metadata about the page - the title, description, URL path, and other settings. It's always between the `---` lines at the top of the file.

Everything *below* the front matter is your page content, written in Markdown.

---

## Step 9: Learn Markdown Basics

Markdown is the simple formatting language you'll use for content. Here's a quick reference:

```markdown
# Big Heading
## Smaller Heading
### Even Smaller

This is a paragraph. Just type normally.

**This is bold text.**
*This is italic text.*

- Bullet point
- Another bullet point

1. Numbered item
2. Another numbered item

[Link text](https://example.com)

![Image description](/assets/images/photo.jpg)
```

That's honestly 90% of what you'll need. For a full reference, see [markdownguide.org/cheat-sheet](https://www.markdownguide.org/cheat-sheet/).

---

## Step 10: Build and Deploy

When you're happy with your site and want to put it on the web:

```bash
npm run build
```

This creates a `_site/` folder containing your finished website. That folder is everything you need to upload to a hosting service.

For a detailed guide on deploying to Netlify, GitHub Pages, or Vercel, see **[[Hosting and Deployment]]**.

---

## What Next?

You've got the basics! Here are good next steps:

- **[[Customisation]]** - Change colours, fonts, logo, and navigation.
- **[[Posts]]** - Start writing blog posts.
- **[[Events]]** - Create your first event.
- **[[Forms]]** - Learn how the contact form works.
- **[[Hosting and Deployment]]** - Get your site live on the web.

> **Tip:** If you ever get stuck or see something you don't understand, copy the file contents into an AI tool (like ChatGPT or GitHub Copilot) and ask it to explain. That's one of the biggest advantages of working with plain text files - they're easy to get help with.
