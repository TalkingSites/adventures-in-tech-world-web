---
title: Forms
---

SpaceClub! includes a reusable form system powered by a single partial (`form.njk`). You define the form's fields in a page's front matter, and the partial generates a complete form with Bootstrap validation, Netlify form submission support, and URL parameter prefilling.

---

## Basic Usage

To add a form to any page, include the form partial and configure it in the front matter:

```markdown
---
layout: base.njk
title: Contact Us
permalink: "/contact/"
formTitle: Get in Touch
formDescription: Have questions? We'd love to hear from you.
formName: contact
formFields:
  - name
  - email
  - message
---

{% include "partials/form.njk" %}
```

This generates a form with Name, Email, and Message fields, complete with validation and a submit button.

---

## Front Matter Properties

| Property | Required | Description |
|---|---|---|
| `formTitle` | Yes | The heading displayed above the form. |
| `formDescription` | No | A short description below the heading. |
| `formName` | Yes | The form's `name` attribute. Used by Netlify to identify the form. Must be unique if you have multiple forms on different pages. |
| `formFields` | Yes | An array of fields. Can be simple strings or objects with more options. |

---

## Defining Fields

### Simple String Format

For basic fields, just list the field names:

```yaml
formFields:
  - name
  - email
  - message
```

The partial applies smart defaults:
- A field named `email` automatically gets `type="email"`.
- A field named `message` automatically renders as a `<textarea>`.
- Everything else defaults to `type="text"`.

### Object Format

For fields that need a specific input type:

```yaml
formFields:
  - name
  - { name: phone, type: tel }
  - { name: age, type: number }
  - email
  - message
```

Available types include any valid HTML input type: `text`, `email`, `tel`, `number`, `url`, `date`, etc.

### Mixing Formats

You can freely mix strings and objects:

```yaml
formFields:
  - name
  - email
  - { name: website, type: url }
  - { name: age, type: number }
  - message
```

---

## How It Works Under the Hood

The `form.njk` partial:

1. Loops through `formFields`.
2. For each field, determines the `name`, input `type`, and whether to render an `<input>` or `<textarea>`.
3. Generates Bootstrap-styled form controls with labels, validation, and feedback messages.
4. Adds Bootstrap client-side validation (the `needs-validation` class).
5. Adds a script that prefills fields from URL query parameters.

---

## URL Prefilling

You can link to a form page with pre-filled values by adding query parameters:

```
https://yoursite.com/contact/?name=Alice&email=alice%40example.com&message=Hello%20there
```

Rules:
- Use the field's `name` as the query key.
- URL-encode special characters (`@` → `%40`, spaces → `%20`).
- Works for both input fields and textareas.

This is useful for creating links from other pages that pre-populate the form - for example, an event page with a "Register" button that prefills the event name.

---

## Netlify Form Submission

> **Note:** Netlify Forms is the only submission method that has been tested with SpaceClub!. The alternatives listed below should work in principle, but they haven't been verified. You may need to adjust the form configuration.

The form includes the `netlify` attribute, which tells Netlify to automatically handle form submissions. When deployed to Netlify:

- Submissions appear in your Netlify dashboard under **Forms**.
- No backend code is needed.
- You can set up email notifications in Netlify for new submissions.

### Using a Different Hosting Provider

If you're not using Netlify, you'll need to handle form submissions yourself. Options include:

- **[Formspree](https://formspree.io)** - A free form backend. Change the `<form>` action to your Formspree URL and remove the `netlify` attribute.
- **[Getform](https://getform.io)** - Similar to Formspree.
- **Custom backend** - Point the form's action to your own API endpoint.

To modify the form, edit `src/_includes/partials/form.njk`. Remove the `netlify` attribute and add your preferred method's `action` attribute.

---

## Creating Multiple Forms

You can have forms on different pages. Just give each one a unique `formName`:

**Contact page:**
```yaml
formName: contact
formFields:
  - name
  - email
  - message
```

**Registration page:**
```yaml
formName: registration
formFields:
  - name
  - email
  - { name: phone, type: tel }
  - { name: dietary-requirements, type: text }
```

Each form appears as a separate form in your Netlify dashboard.

---

## Example: RSVP Form

```markdown
---
layout: base.njk
title: RSVP
permalink: "/rsvp/"
formTitle: RSVP to Our Event
formDescription: Let us know you're coming!
formName: rsvp
formFields:
  - name
  - email
  - { name: guests, type: number }
  - message
---

{% include "partials/form.njk" %}
```
