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
