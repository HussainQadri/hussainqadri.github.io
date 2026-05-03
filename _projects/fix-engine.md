---
title: FIX Engine
permalink: /projects/fix-engine/
project: fix-engine
image: /images/fix-engine-card.png
summary: "An educational C++ FIX engine built to understand how FIX messages are parsed, validated, typed, and eventually handled over a session. It currently focuses on tag=value parsing, BodyLength/CheckSum checks, FIX 4.2 dictionary validation, and typed admin-message wrappers."
stack: C++
accent: "#f38ba8"
order: 1
---

# FIX Engine

A C++ FIX engine focused on correctness, validation, and protocol learning. 

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
