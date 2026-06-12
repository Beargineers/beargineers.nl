---
title: Learn
permalink: /learn/
nav_title: Learn
nav_order: 2
published: false
eyebrow: Knowledge base
subtitle: Curated paths for FTC programming, CAD, robot design, and manufacturing.
---

This section is for students who want to become useful quickly on an FTC team. The goal is not to collect every possible tutorial, but to recommend the materials Beargineers would actually use for onboarding and technical growth.

<div class="path-grid">
  <section class="path-card">
    <p class="card-meta">Software path</p>
    <h2>Kotlin for FTC</h2>
    <p>Start with Kotlin fundamentals, then move into FTC SDK structure, op modes, command design, autonomous routines, sensors, and robot state.</p>
  </section>
  <section class="path-card">
    <p class="card-meta">Design path</p>
    <h2>Fusion 360 for FTC</h2>
    <p>Learn sketches, constraints, assemblies, vendor part workflows, design for 3D printing, and mechanism documentation.</p>
  </section>
  <section class="path-card">
    <p class="card-meta">Robot path</p>
    <h2>FTC engineering basics</h2>
    <p>Build practical vocabulary around drivetrains, intakes, shooters, localization, vision, wiring, and iterative testing.</p>
  </section>
</div>

## Resource library

<div class="filter-note">Use the tags and level labels to pick the next useful thing, not the most impressive thing.</div>

<div class="listing-grid">
{% assign resources = site.resources | sort: "order" %}
{% for item in resources %}
  <article class="listing-card">
    <p class="card-meta">{{ item.area | capitalize }}{% if item.level %} · {{ item.level | capitalize }}{% endif %}</p>
    <h3><a href="{{ item.url | relative_url }}">{{ item.title }}</a></h3>
    <p>{{ item.summary }}</p>
    {% if item.tags %}
    <p class="tag-row">
      {% for tag in item.tags %}
      <span>{{ tag }}</span>
      {% endfor %}
    </p>
    {% endif %}
  </article>
{% endfor %}
</div>
