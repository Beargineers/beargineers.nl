---
title: Engineering Portfolios
permalink: /portfolio/
nav_title: Portfolio
nav_order: 6
eyebrow: Season documents
subtitle: Downloadable engineering portfolios and the web pages that support them.
---

The portfolio is still important, but it is now one output of the team's documentation system rather than the structure of the whole website.

<div class="listing-grid">
{% assign portfolios = site.portfolios | sort: "season" | reverse %}
{% for item in portfolios %}
  <article class="listing-card">
    <p class="card-meta">{{ item.season }} · {{ item.game }}</p>
    <h2><a href="{{ item.url | relative_url }}">{{ item.title }}</a></h2>
    <p>{{ item.summary }}</p>
    <p class="button-row compact">
      <a class="button primary" href="{{ item.pdf_url | relative_url }}" target="_blank" rel="noopener">Download PDF</a>
      <a class="button" href="{{ item.url | relative_url }}">Highlights</a>
    </p>
  </article>
{% endfor %}
</div>
