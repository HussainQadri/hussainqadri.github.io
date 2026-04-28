---
title: Rabab Tuner
permalink: /projects/rabab-tuner/
project: rabab-tuner
summary: "A real-time browser-based tuner for the Rabab, built with React and Python/Flask. The frontend captures live microphone audio via the Web Audio API and streams ~200ms PCM chunks over WebSockets to a backend running a custom YIN pitch detection algorithm. The engine calculates cent-deviation against the Sargam scale, manages backpressure to prevent frame drops, and drives an analog-style gauge UI with cent-level precision."
image: /images/rabab-wireframe.png
stack: React + Flask
accent: "#94e2d5"
order: 2
---

# Rabab Tuner

A browser-based tuner using recorded audio and frequency detection.

## Overview

When I was spending time with my teacher learning this instrument, I noticed how hard it is to tune it. It's based on a non-classical musical notation called sargam - there are mappings to the classical notes but its not standardised. To the beginner, this would all seem very alien. I wanted to make this experience a little bit easier.

[See me playing this instrument](/projects/rabab-tuner/gallery/)

## Updates

{% assign project_posts = site.posts | where_exp: "post", "post.project == page.project" %}
{% if project_posts.size > 0 %}
  {% for post in project_posts %}
## {{ post.title }}
<span class="post-date">{{ post.date | date: "%d %b %Y" }}</span>

{{ post.content }}
{% unless forloop.last %}<hr class="post-separator">{% endunless %}
  {% endfor %}
{% else %}
<p>No updates yet.</p>
{% endif %}
