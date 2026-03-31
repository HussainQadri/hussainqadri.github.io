---
title: Rabab Tuner
permalink: /projects/rabab-tuner/
project: rabab-tuner
summary: A browser-based tuner using recorded audio and frequency detection.
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
  {% assign project_posts_reversed = project_posts | reverse %}
  {% for post in project_posts_reversed %}
## {{ post.title }}
<span class="post-date">{{ post.date | date: "%d %b %Y" }}</span>

{{ post.content }}
  {% endfor %}
{% else %}
<p>No updates yet.</p>
{% endif %}
