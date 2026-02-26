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
{% set link = "https://adventures-in-tech.world/" %}
{% set icon = "play-fill" %}
{% set style = "primary" %}
{% set align = "start" %}
{% include "partials/btn.njk" %}

{% set text = "Dev Blog" %}
{% set link = "/posts/" %}
{% set icon = "newspaper" %}
{% set style = "outline-secondary" %}
{% include "partials/btn.njk" %}
</div>

<!-- <div class="my-5 rounded-3 overflow-hidden border border-primary border-opacity-25 shadow-lg" id="play">
  <img src="/assets/images/gameplay.gif" alt="Adventures in Tech World gameplay preview" class="img-fluid w-100 d-block">
</div> -->

{% include "partials/featureCards.njk" %}

<div class="text-center mt-4 mb-5">
{% set text = "Learn More About the Game" %}
{% set link = "/about/" %}
{% set icon = "arrow-right" %}
{% set style = "outline-secondary" %}
{% set align = "center" %}
{% include "partials/btn.njk" %}
</div>

## Latest from the Dev Blog

Follow the journey as we build Adventures in Tech World.

{% set type = "preview" %}
{% include "partials/postsList.njk" %}
