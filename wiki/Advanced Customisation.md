---
title: Advanced Customisation
---

This guide is for when you want to go beyond editing content and build your own custom list components. Two common scenarios:

1. **Cards from a JSON file** - You have a list of items (menu, team, services, products) that don't need individual pages.
2. **Cards from Markdown files** - Each item in your list needs its own full page (like how posts and events work).

---

## Option 1: Cards from a JSON Data File

This is the simpler approach. Good for things like a menu, a list of services, team members, FAQs, or anything that doesn't need its own URL.

### Step 1: Create the Data File

Create a JSON file in `src/_data/`. The filename becomes the variable name available in all your templates.

For example, `src/_data/menu.json`:

```json
[
  {
    "name": "Flat White",
    "description": "Strong espresso with velvety steamed milk.",
    "price": "$5.50",
    "category": "Coffee"
  },
  {
    "name": "Avocado Toast",
    "description": "Sourdough with fresh avocado and sea salt.",
    "price": "$14.00",
    "category": "Food"
  }
]
```

### Step 2: Create the Card Partial

Create a new partial in `src/_includes/partials/`. For example, `src/_includes/partials/menuCards.njk`:

```nunjucks
{% for item in menu %}
<div class="col-md-4 mb-4">
  <div class="card border-primary border-opacity-25 shadow-sm h-100">
    <div class="card-body d-flex flex-column">
      <div class="mb-2">
        <span class="badge bg-primary bg-opacity-10 text-primary border border-primary border-opacity-25 rounded-pill px-3 py-1 small">
          {{ item.category }}
        </span>
      </div>
      <h5 class="card-title">{{ item.name }}</h5>
      <p class="card-text text-muted small flex-grow-1">{{ item.description }}</p>
      <p class="fw-bold text-primary mt-2 mb-0">{{ item.price }}</p>
    </div>
  </div>
</div>
{% endfor %}
```

The variable name in `{% for item in menu %}` matches the JSON filename (`menu.json` → `menu`).

### Step 3: Use It in a Page

Create `src/menu.md` (or add it to any existing page):

```markdown
---
layout: base.njk
title: Our Menu
permalink: "/menu/"
---

# Our Menu

<div class="row g-4">
{% include "partials/menuCards.njk" %}
</div>
```

### Step 4: Add to Navigation (Optional)

Add an entry to `src/_data/navbar.json`:

```json
{
  "title": "Menu",
  "href": "/menu",
  "footer": false
}
```

That's it. Edit `menu.json` to add, remove, or change items and the page updates on the next build.

---

## Option 2: Cards from Markdown Files in a Folder

This approach mirrors how posts and events work. Each item has its own page and its own URL. Good for team profiles, project portfolios, articles, or any content that deserves a dedicated page.

### Step 1: Create the Content Folder

Create a new folder in `src/`. For example, `src/team/`.

### Step 2: Create a Directory Data File

Create `src/team/team.json` to set defaults for all files in this folder:

```json
{
  "layout": "base.njk"
}
```

This means every `.md` file in `src/team/` will automatically use `base.njk` as its layout without you having to specify it in every file.

### Step 3: Create the Content Files

Create one `.md` file per item. For example, `src/team/alice.md`:

```markdown
---
title: Alice Chen
role: Lead Designer
bio: Alice has been designing interfaces for 10 years.
image: /assets/images/team/alice.jpg
---

Alice joined us in 2021 and has led the visual direction of every project since.
```

And `src/team/bob.md`:

```markdown
---
title: Bob Martinez
role: Developer
bio: Bob builds things that work and fixes things that don't.
image: /assets/images/team/bob.jpg
---

Bob has been part of the team since the beginning.
```

### Step 4: Register a Collection in `.eleventy.js`

Open `.eleventy.js` and add a new collection inside `module.exports = function (eleventyConfig) {`:

```js
eleventyConfig.addCollection("team", function (collectionApi) {
  return collectionApi.getFilteredByGlob("src/team/*.md");
});
```

Place it near the other `addCollection` calls.

### Step 5: Create the Card Partial

Create `src/_includes/partials/teamCards.njk`:

```nunjucks
{% if collections.team and collections.team | length > 0 %}
<div class="row g-4">
{% for member in collections.team %}
<div class="col-md-4">
  <div class="card border-primary border-opacity-25 shadow-sm h-100 overflow-hidden">
    {% if member.data.image %}
    <div style="height: 200px; overflow: hidden;">
      <img src="{{ member.data.image }}" class="img-fluid w-100 h-100 object-fit-cover" alt="{{ member.data.title }}">
    </div>
    {% endif %}
    <div class="card-body d-flex flex-column">
      <h5 class="card-title">{{ member.data.title }}</h5>
      {% if member.data.role %}
      <p class="text-primary small fw-semibold mb-2">{{ member.data.role }}</p>
      {% endif %}
      {% if member.data.bio %}
      <p class="card-text text-muted small flex-grow-1">{{ member.data.bio }}</p>
      {% endif %}
      <div class="mt-auto">
        <a href="{{ member.url }}" class="btn btn-outline-primary btn-sm w-100 stretched-link">Read More</a>
      </div>
    </div>
  </div>
</div>
{% endfor %}
</div>
{% else %}
<p class="text-muted">No team members found.</p>
{% endif %}
```

### Step 6: Create the Listing Page

Create `src/team/team.njk`:

```nunjucks
---
layout: base.njk
title: Our Team
permalink: "/team/"
---

<h1>Our Team</h1>

{% include "partials/teamCards.njk" %}
```

### Step 7: Add to Navigation (Optional)

```json
{
  "title": "Team",
  "href": "/team",
  "footer": false
}
```

Now each team member has their own page at `/team/alice/` and `/team/bob/`, and the `/team/` listing page shows all of them as cards.

---

## Hiding Template Files

If you create a `template.md` inside your new folder as a starting point for new entries, rename it to `template.njk`. The collection glob (`src/team/*.md`) only picks up `.md` files, so `template.njk` won't appear in the listing. See [[Events]] and [[Posts]] for the same pattern.

---

## Which Approach Should I Use?

| Use JSON (`_data/`) | Use Markdown files |
|---|---|
| Items don't need their own pages | Each item needs its own full page and URL |
| Data is simple and list-like | Items have long-form content |
| You'll update it frequently | Updates are occasional |
| Menu, services, FAQs, testimonials | Team bios, projects, case studies, articles |
