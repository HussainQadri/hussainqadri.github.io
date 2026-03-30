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
<ul class="clean-list">
  {% for post in project_posts %}
    <li>
      <strong><a href="{{ post.url | relative_url }}">{{ post.title }}</a></strong><br>
      {{ post.date | date: "%d %b %Y" }}
    </li>
  {% endfor %}
</ul>
{% else %}
<p>No updates yet.</p>
{% endif %}
