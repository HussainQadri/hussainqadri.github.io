---
title: Sift
permalink: /projects/sift/
project: sift
image: /images/sift-card.png
summary: "A Rust semantic code search CLI that uses Tree-sitter to extract functions, embeds code with a pretrained model, and ranks functions against natural-language queries with vector similarity."
stack: Rust
accent: "#dea584"
order: 4
---

# Sift

A Rust semantic search tool for finding code by meaning rather than exact keyword matches.

## Overview

The current prototype parses Rust, Python, and C++ files with Tree-sitter, extracts functions, embeds their source text, and ranks them against a natural-language query using cosine similarity. The search output stays compact by showing function headers while the richer full function source is used for retrieval.

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
