---
title: Engineering Portfolios
permalink: /portfolio/
nav_title: Portfolio
nav_order: 6
eyebrow: Season documents
---

An **FTC Engineering Portfolio** is a concise document that tells the story of a team’s season: who the team is, what goals it set, how it designed, built, programmed, tested, and improved the robot, and how it contributed to the wider FTC community. Judges use it as a key source of evidence for judged awards, so it should clearly show the engineering design process, major robot decisions, iterations, failures and fixes, outreach, team organization, and future plans. In practice, it is not just a scrapbook of everything the team did, but a curated argument for why the team’s work was thoughtful, well-documented, impactful, and award-worthy.

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
