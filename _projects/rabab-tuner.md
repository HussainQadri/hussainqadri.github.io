---
title: Rabab Tuner
permalink: /projects/rabab-tuner/
project: rabab-tuner
summary: A browser-based tuner using recorded audio and frequency detection.
order: 2
---

# Rabab Tuner

A browser-based tuner using recorded audio and frequency detection.

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
