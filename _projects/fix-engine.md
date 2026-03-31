---
title: FIX Engine
permalink: /projects/fix-engine/
project: fix-engine
summary: A C++ FIX engine focused on correctness, validation, and protocol learning.
order: 1
---

# FIX Engine

A C++ FIX engine focused on correctness, validation, and protocol learning.

## Updates

{% assign project_posts = site.posts | where_exp: "post", "post.project == page.project" %}
{% if project_posts.size > 0 %}
  {% assign project_posts_reversed = project_posts | reverse %}
  {% for post in project_posts_reversed %}
### {{ post.title }}
<span class="post-date">{{ post.date | date: "%d %b %Y" }}</span>

{{ post.content }}
  {% endfor %}
{% else %}
<p>No updates yet.</p>
{% endif %}
