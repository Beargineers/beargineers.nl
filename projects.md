---
title: Projects
permalink: /projects/
nav_title: Projects
nav_order: 3
published: false
eyebrow: Open engineering
subtitle: Team-built code, libraries, examples, and utilities that other FTC teams can reuse.
---

This is the public shelf for software and engineering tools that Beargineers develops. Project pages should explain what the thing is for, whether it is actively maintained, how to try it, and where the source lives.

<div class="listing-grid">
{% assign projects = site.projects | sort: "order" %}
{% for item in projects %}
  <article class="listing-card">
    <p class="card-meta">{{ item.status | default: "Project" }}</p>
    <h2><a href="{{ item.url | relative_url }}">{{ item.title }}</a></h2>
    <p>{{ item.summary }}</p>
    <p class="button-row compact">
      <a class="button" href="{{ item.url | relative_url }}">Read notes</a>
      {% if item.github_url %}<a class="button" href="{{ item.github_url }}" target="_blank" rel="noopener">GitHub</a>{% endif %}
    </p>
  </article>
{% endfor %}
</div>
