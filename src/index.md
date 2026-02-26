---
layout: index.njk
title: Adventures in Tech World
description: Adventures in Tech World is an educational multiplayer 2D pixel game where players explore, solve coding challenges, and collaborate in real time. Play free in your browser.
permalink: "/"
---

# Adventures in Tech World

**A multiplayer pixel world where tech people gather.** Explore unique maps, tackle coding challenges, chat with an AI tutor, and connect with other developers via real-time voice and video. No download needed.

<div class="d-flex flex-wrap gap-3 my-4">
{% set text = "Play in Your Browser" %}
{% set link = "#play" %}
{% set icon = "play-fill" %}
{% include "partials/btn.njk" %}

{% set text = "Dev Blog" %}
{% set link = "/posts/" %}
{% set icon = "newspaper" %}
{% set style = "outline-secondary" %}
{% include "partials/btn.njk" %}
</div>

<div class="my-5 rounded-3 overflow-hidden border border-primary border-opacity-25 shadow-lg" id="play">
  <img src="/assets/images/gameplay.gif" alt="Adventures in Tech World gameplay preview" class="img-fluid w-100 d-block">
</div>

{% include "partials/featureCards.njk" %}

## The World

Adventures in Tech World is set across **6 unique maps**: Open Arena, The L-Room, Four Corners, Simple Maze, The Library, and The Workshop. Each map is fully explorable with wall occlusion, meaning your character walks behind walls just like a classic RPG.

{% include "partials/gameDetails.njk" %}

## Clawd - Your AI Tutor

Every player has access to **Clawd**, an in-game AI tutor powered by Claude. Ask questions, get code hints, and receive instant feedback on your solutions without ever leaving the game world. Clawd also supports voice input and output for a truly hands-free learning experience.

## Open Source

Adventures in Tech World is **open source**. The game code is freely available to study, fork, and contribute to. We believe that an open codebase is especially powerful for an educational coding game - players can literally read the code behind the game they are playing. Being open source also invites the developer community to help shape the project, while we build a sustainable business around hosting, access, and extended features.

<div class="d-flex flex-wrap gap-3 my-4">
{% set text = "View Game on GitHub" %}
{% set link = site.gameRepo %}
{% set icon = "github" %}
{% set style = "outline-primary" %}
{% set align = "start" %}
{% include "partials/btn.njk" %}
{% set text = "Get in Touch" %}
{% set link = "/contact/" %}
{% set icon = "envelope" %}
{% set style = "outline-secondary" %}
{% set align = "start" %}
{% include "partials/btn.njk" %}
</div>

## The Vision

Adventures in Tech World is an educational prototype being developed with support from Screen Australia's Games Production Fund. Our goal is to grow it from a working prototype into a fully featured game - an accessible, engaging space where people can discover programming through play, together.

## Built With

<div class="d-flex flex-wrap gap-2 my-3">
  <span class="badge rounded-pill bg-primary bg-opacity-25 text-primary border border-primary border-opacity-25 px-3 py-2">Flutter</span>
  <span class="badge rounded-pill bg-secondary bg-opacity-25 text-secondary border border-secondary border-opacity-25 px-3 py-2">Flame Engine</span>
  <span class="badge rounded-pill bg-primary bg-opacity-25 text-primary border border-primary border-opacity-25 px-3 py-2">LiveKit</span>
  <span class="badge rounded-pill bg-secondary bg-opacity-25 text-secondary border border-secondary border-opacity-25 px-3 py-2">Claude API</span>
  <span class="badge rounded-pill bg-primary bg-opacity-25 text-primary border border-primary border-opacity-25 px-3 py-2">Firebase</span>
</div>

## Latest from the Dev Blog

Follow the journey as we build Adventures in Tech World.

{% set type = "preview" %}
{% include "partials/postsList.njk" %}
