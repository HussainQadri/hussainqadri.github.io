---
title: EncodeEd
permalink: /projects/encodeed/
project: encodeed
summary: A compression visualisation project for algorithms like Huffman and LZ77.
order: 3
---

# EncodeEd

A compression visualisation project for algorithms like Huffman and LZ77.

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
