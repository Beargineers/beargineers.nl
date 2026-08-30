---
title: Learn
permalink: /learn/
nav_title: Learn
nav_order: 2
layout: docs
published: true
eyebrow: Knowledge base
subtitle: Folder-based FTC documentation for programming, CAD, robot design, and manufacturing.
---

This section is for students who want to become useful quickly on an FTC team. The goal is not to collect every possible tutorial, but to recommend the materials Beargineers would actually use for onboarding and technical growth.

The notes here are working documentation: practical patterns, team conventions, and examples that should help students make better engineering decisions during a season.

## Continue learning

{% assign published_resources = site.resources | where_exp: "resource", "resource.published != false" | sort: "path" %}
{% if published_resources.size > 0 %}
<ul>
  {% for resource in published_resources %}
    <li><a href="{{ resource.url | relative_url }}">{{ resource.title }}</a>{% if resource.summary %}: {{ resource.summary }}{% endif %}</li>
  {% endfor %}
</ul>
{% endif %}
