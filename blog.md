---
title: Blog
permalink: /blog/
nav_title: Blog
nav_order: 5
published: false
eyebrow: Team stories
subtitle: Build logs, competition reports, engineering lessons, outreach notes, and season retrospectives.
---

The blog is for stories that should not be trapped inside a portfolio PDF. Short posts are fine when they capture a decision, a failure, a test result, or an event while the details are still fresh.

<div class="listing-grid">
{% assign posts = site.posts | sort: "date" | reverse %}
{% for post in posts %}
  <article class="listing-card">
    <p class="card-meta">{{ post.date | date: "%b %-d, %Y" }}{% if post.category %} · {{ post.category }}{% endif %}</p>
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    <p>{{ post.excerpt | strip_html | truncate: 180 }}</p>
  </article>
{% endfor %}
</div>
