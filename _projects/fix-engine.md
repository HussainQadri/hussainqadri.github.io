---
title: FIX Engine
permalink: /projects/fix-engine/
project: fix-engine
summary: "This repository features a custom-built FIX (Financial Information eXchange) Protocol Engine, developed in C++, that is designed to decode raw, standard financial messaging streams into structured FIX messages. Built entirely from scratch without reliance on third-party parsing libraries, the engine demonstrates systems-level engineering with a strong focus on robust TCP/IP networking, complex string manipulation, and strict protocol handling. It is engineered to ingest continuous live byte streams, seamlessly manage advanced edge cases like TCP message fragmentation, and execute rigorous on-the-fly checksum validations to ensure data integrity across millions of parsed trading messages."
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
