---
layout: portfolio
title: Beargineers
nav_title: Home
nav_order: 1
eyebrow: FTC Team #27628
subtitle: FTC team archive, match records, and season portfolio downloads.
---

<section class="hero-panel">
  <div>
    <p class="section-kicker">International School of Amsterdam</p>
    <h2>Build better robots, and leave better starting points for the next team.</h2>
    <p>Beargineers are turning this site into a public engineering archive. This work is still in progress. For now, the site focuses on team information, match records, and downloadable engineering portfolios while resource sections are still being prepared.</p>
    <p class="button-row">
      <a class="button" href="{{ '/portfolio/' | relative_url }}">Download portfolio</a>
      <a class="button" href="{{ '/matches/' | relative_url }}">Match archive</a>
    </p>
  </div>
  <img src="{{ '/assets/images/gamma.webp' | relative_url }}" alt="Beargineers Gamma robot">
</section>

{% assign learn_page = site.pages | where: "url", "/learn/" | first %}
{% assign projects_page = site.pages | where: "url", "/projects/" | first %}
{% assign cad_page = site.pages | where: "url", "/cad/" | first %}
{% assign blog_page = site.pages | where: "url", "/blog/" | first %}
{% if learn_page or projects_page or cad_page or blog_page %}
<section class="feature-grid">
  {% if learn_page %}
  <a class="feature-card" href="{{ '/learn/' | relative_url }}">
    <span>Learn</span>
    <h3>Kotlin, FTC SDK, autonomous thinking, Fusion 360, and manufacturing basics.</h3>
  </a>
  {% endif %}
  {% if projects_page %}
  <a class="feature-card" href="{{ '/projects/' | relative_url }}">
    <span>Projects</span>
    <h3>Open-source libraries, utilities, examples, and robot software patterns.</h3>
  </a>
  {% endif %}
  {% if cad_page %}
  <a class="feature-card" href="{{ '/cad/' | relative_url }}">
    <span>CAD</span>
    <h3>Reusable robot mechanisms, printable parts, STEP files, and design notes.</h3>
  </a>
  {% endif %}
  {% if blog_page %}
  <a class="feature-card" href="{{ '/blog/' | relative_url }}">
    <span>Blog</span>
    <h3>Competition stories, engineering decisions, lessons learned, and outreach notes.</h3>
  </a>
  {% endif %}
</section>
{% endif %}

{% assign featured_resources = site.resources | where: "featured", true | sort: "order" %}
{% if featured_resources.size > 0 %}

## Featured resources

<div class="listing-grid">
{% for item in featured_resources limit: 3 %}
  <article class="listing-card">
    <p class="card-meta">{{ item.area | capitalize }}{% if item.level %} · {{ item.level | capitalize }}{% endif %}</p>
    <h3><a href="{{ item.url | relative_url }}">{{ item.title }}</a></h3>
    <p>{{ item.summary }}</p>
  </article>
{% endfor %}
</div>
{% endif %}

{% assign featured_projects = site.projects | where: "featured", true | sort: "order" %}
{% assign featured_cad = site.cad | where: "featured", true | sort: "order" %}
{% if featured_projects.size > 0 or featured_cad.size > 0 %}
## Latest from the archive

<div class="listing-grid">
  {% for item in featured_projects limit: 2 %}
  <article class="listing-card">
    <p class="card-meta">Project</p>
    <h3><a href="{{ item.url | relative_url }}">{{ item.title }}</a></h3>
    <p>{{ item.summary }}</p>
  </article>
  {% endfor %}
  {% for item in featured_cad limit: 1 %}
  <article class="listing-card">
    <p class="card-meta">CAD drop</p>
    <h3><a href="{{ item.url | relative_url }}">{{ item.title }}</a></h3>
    <p>{{ item.summary }}</p>
  </article>
  {% endfor %}
</div>
{% endif %}
