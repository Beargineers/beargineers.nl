---
title: CAD & Downloads
permalink: /cad/
nav_title: CAD
nav_order: 4
published: false
eyebrow: Design archive
subtitle: Reusable designs, printable parts, CAD files, and design notes from Beargineers robots.
---

CAD drops should be small enough to understand and reuse. A useful drop includes the design intent, the file format, season/game context, manufacturing notes, and a warning when the design was experimental.

<div class="listing-grid">
{% assign cad_items = site.cad | sort: "order" %}
{% for item in cad_items %}
  <article class="listing-card">
    <p class="card-meta">{{ item.season }}{% if item.mechanism %} · {{ item.mechanism }}{% endif %}</p>
    <h2><a href="{{ item.url | relative_url }}">{{ item.title }}</a></h2>
    <p>{{ item.summary }}</p>
    <p class="button-row compact">
      <a class="button" href="{{ item.url | relative_url }}">Design notes</a>
      {% if item.download_url %}<a class="button" href="{{ item.download_url | relative_url }}">Download</a>{% endif %}
    </p>
  </article>
{% endfor %}
</div>
