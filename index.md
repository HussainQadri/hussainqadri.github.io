---
title: Home
permalink: /
---

{% assign visible_projects = site.projects | where_exp: "project", "project.listed != false" %}
{% assign pinned_projects = visible_projects | where: "pinned", true | sort: "order" %}
{% assign other_projects = visible_projects | where_exp: "project", "project.pinned != true" | sort: "order" %}
{% assign sorted_projects = pinned_projects | concat: other_projects %}
{% assign latest_post = site.posts | first %}
{% assign recent_quests = site.side_quests | where_exp: "quest", "quest.listed != false" | sort: "date" | reverse %}

<section class="home-intro">
  <div class="home-intro-copy">
    <h1>Hussain Qadri</h1>
    <p>I am a Computer Science student at King's College London interested in systems programming, developer tools, information retrieval, and building technical projects from first principles.</p>
    <div class="home-intro-links">
      {% if latest_post %}<a href="{{ latest_post.url | relative_url }}">Read the latest update</a>{% endif %}
      <a href="{{ '/projects/' | relative_url }}">Browse projects</a>
    </div>
  </div>
</section>

<section class="home-section">
  <header class="section-head">
    <h2>Recent writing</h2>
  </header>

  <ol class="post-list">
    {% for post in site.posts limit: 6 %}
      {% assign project = site.projects | where: "project", post.project | first %}
      <li>
        <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%d %b %Y" }}</time>
        <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
        <span>{{ project.title | default: post.project }}</span>
      </li>
    {% endfor %}
  </ol>
</section>

<section class="home-section">
  <header class="section-head">
    <h2>Projects</h2>
    <a href="{{ '/projects/' | relative_url }}">All projects</a>
  </header>

  <div class="card-grid card-grid-2">
    {% for project in sorted_projects limit: 4 %}
      <a class="card" href="{{ project.url | relative_url }}">
        {% if project.image %}<img src="{{ project.image | relative_url }}" alt="{{ project.title }} project preview" loading="lazy">{% endif %}
        <div class="card-body">
          {% if project.stack %}<span class="card-topic">{{ project.stack }}</span>{% endif %}
          <strong>{{ project.title }}</strong>
          <p>{{ project.summary }}</p>
        </div>
      </a>
    {% endfor %}
  </div>
</section>

<section class="home-section">
  <header class="section-head">
    <h2>Side-quests</h2>
    <a href="{{ '/side-quests/' | relative_url }}">All side-quests</a>
  </header>

  <div class="card-grid card-grid-3">
    {% for quest in recent_quests limit: 3 %}
      {% assign topic = site.data.topics[quest.topic] %}
      <a class="card" href="{{ quest.url | relative_url }}">
        {% if quest.image %}<img src="{{ quest.image | relative_url }}" alt="{{ quest.title }} preview" loading="lazy">{% endif %}
        <div class="card-body">
          <span class="card-topic">{{ topic.label | default: quest.topic }}</span>
          <strong>{{ quest.title }}</strong>
          <p>{{ quest.summary }}</p>
        </div>
      </a>
    {% endfor %}
  </div>
</section>

<section class="home-section home-personal">
  <figure>
    <img src="{{ '/assets/images/projects/rabab-tuner/playing-rabab-pic1.jpeg' | relative_url }}" alt="Hussain playing the rabab" loading="lazy">
  </figure>
  <div>
    <header class="section-head">
      <h2>Away from the keyboard</h2>
    </header>
    <ul>
      <li>I play the traditional Afghan rabab and hold Grade 6 in Music Theory.</li>
      <li>I hold HSK 3 certification in Mandarin and B1 in French.</li>
      <li>I am an open-water certified scuba diver.</li>
    </ul>
  </div>
</section>
