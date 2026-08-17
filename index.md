---
title: Home
permalink: /
---

{% assign visible_projects = site.projects | where_exp: "project", "project.listed != false" %}
{% assign pinned_projects = visible_projects | where: "pinned", true | sort: "order" %}
{% assign other_projects = visible_projects | where_exp: "project", "project.pinned != true" | sort: "order" %}
{% assign sorted_projects = pinned_projects | concat: other_projects %}
{% assign latest_post = site.posts | first %}
{% assign latest_project = site.projects | where: "project", latest_post.project | first %}
{% assign recent_quests = site.side_quests | where_exp: "quest", "quest.listed != false" | sort: "date" | reverse %}

<section class="home-hero">
  <div class="home-hero-copy">
    <h1>Hussain Qadri</h1>
    <p class="home-dek">Systems, software, and side projects.</p>
    <p class="home-intro">I am a Computer Science student at King's College London interested in systems programming, developer tools, information retrieval, and building technical projects from first principles.</p>

    <dl class="current-work" aria-label="Current focus">
      <div><dt>search</dt><dd>Sift in Rust with Tree-sitter and a custom HNSW index</dd></div>
      <div><dt>protocols</dt><dd>A C++ FIX 4.2 engine with validation and session state</dd></div>
      <div><dt>audio</dt><dd>A browser-based rabab tuner using YIN and streamed PCM</dd></div>
    </dl>

    <div class="hero-links">
      {% if latest_post %}<a href="{{ latest_post.url | relative_url }}">Read the latest update</a>{% endif %}
      <a href="{{ '/projects/' | relative_url }}">Browse projects</a>
    </div>
  </div>

  <div class="home-artifacts" aria-label="Selected project previews">
    <figure class="artifact artifact-sift">
      <a href="{{ '/projects/sift/' | relative_url }}"><img src="{{ '/images/sift-card.png' | relative_url }}" alt="Sift semantic search project preview"></a>
      <figcaption><strong>Sift</strong><span>Semantic code search</span></figcaption>
    </figure>
    <figure class="artifact artifact-fix">
      <a href="{{ '/projects/fix-engine/' | relative_url }}"><img src="{{ '/images/fix-engine-card.png' | relative_url }}" alt="FIX Engine project preview"></a>
      <figcaption><strong>FIX Engine</strong><span>Protocol engineering</span></figcaption>
    </figure>
    <figure class="artifact artifact-plotlyst">
      <a href="{{ '/side-quests/plotlyst/' | relative_url }}"><img src="{{ '/images/plotlyst.png' | relative_url }}" alt="Plotlyst chart editor preview"></a>
      <figcaption><strong>Plotlyst</strong><span>Business chart editor</span></figcaption>
    </figure>
  </div>
</section>

<section class="home-writing" data-reveal>
  <header class="section-heading">
    <p>Latest writing</p>
    <h2>Notes from work in progress.</h2>
  </header>

  <div class="writing-composition">
    {% if latest_post %}
      <div class="featured-writing">
        {% if latest_project.image %}
          <a class="featured-writing-image" href="{{ latest_post.url | relative_url }}">
            <img src="{{ latest_project.image | relative_url }}" alt="{{ latest_project.title }} project preview">
          </a>
        {% endif %}
        <p class="featured-writing-project">{{ latest_project.title | default: latest_post.project }}</p>
        <h3><a href="{{ latest_post.url | relative_url }}">{{ latest_post.title }}</a></h3>
        <p>{{ latest_post.excerpt | strip_html | strip_newlines | truncate: 190 }}</p>
        <a class="text-link" href="{{ latest_post.url | relative_url }}">Read this update</a>
      </div>
    {% endif %}

    <div class="writing-ledger">
      {% for post in site.posts limit: 5 %}
        {% assign project = site.projects | where: "project", post.project | first %}
        <article class="writing-row">
          <div class="writing-row-meta">
            <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%d %b %Y" }}</time>
            <span>{{ project.title | default: post.project }}</span>
          </div>
          <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
          <p>{{ post.excerpt | strip_html | strip_newlines | truncate: 145 }}</p>
        </article>
      {% endfor %}
    </div>
  </div>
</section>

<section class="home-projects" data-reveal>
  <header class="section-heading section-heading-inline">
    <div>
      <p>Projects</p>
      <h2>Things I am building to understand.</h2>
    </div>
    <a class="text-link" href="{{ '/projects/' | relative_url }}">View all projects</a>
  </header>

  <div class="project-showcase">
    {% for project in sorted_projects limit: 4 %}
      <a class="project-feature project-feature-{{ forloop.index }}" href="{{ project.url | relative_url }}" style="--project-accent: {{ project.accent | default: '#315f73' }}">
        {% if project.image %}<img src="{{ project.image | relative_url }}" alt="{{ project.title }} project preview">{% endif %}
        <div>
          <span>{{ project.stack }}</span>
          <h3>{{ project.title }}</h3>
          <p>{{ project.summary }}</p>
        </div>
      </a>
    {% endfor %}
  </div>
</section>

<section class="home-personal" data-reveal>
  <figure>
    <img src="{{ '/assets/images/projects/rabab-tuner/playing-rabab-pic1.jpeg' | relative_url }}" alt="Hussain playing the rabab">
    <figcaption>Playing the rabab</figcaption>
  </figure>
  <div class="personal-copy">
    <p>Away from the keyboard</p>
    <h2>Music, languages, and the occasional dive.</h2>
    <ul>
      <li>I play the traditional Afghan rabab and hold Grade 6 in Music Theory.</li>
      <li>I hold HSK 3 certification in Mandarin and B1 in French.</li>
      <li>I am an open-water certified scuba diver, and I love <a href="https://store.steampowered.com/app/1422450/Deadlock/">Deadlock</a>.</li>
    </ul>
  </div>
</section>

<section class="home-quests" data-reveal>
  <header class="section-heading section-heading-inline">
    <div>
      <p>Side-quests</p>
      <h2>Interests that became small projects.</h2>
    </div>
    <a class="text-link" href="{{ '/side-quests/' | relative_url }}">View all side-quests</a>
  </header>

  <div class="quest-showcase">
    {% for quest in recent_quests limit: 3 %}
      {% assign topic = site.data.topics[quest.topic] %}
      <a class="quest-feature quest-feature-{{ forloop.index }}" href="{{ quest.url | relative_url }}">
        {% if quest.image %}<img src="{{ quest.image | relative_url }}" alt="{{ quest.title }} preview">{% endif %}
        <div>
          <span>{{ topic.label | default: quest.topic }}</span>
          <h3>{{ quest.title }}</h3>
          <p>{{ quest.summary }}</p>
        </div>
      </a>
    {% endfor %}
  </div>
</section>
