---
layout: portfolio
title: Official matches
permalink: /matches/
nav_title: Matches
nav_order: 9
eyebrow: Competition archive
subtitle: Recordings of the matches our team played
---

Match archives are organized by season, then by event. Each event card includes official FIRST links, video timestamps, qualification results, playoff results, and short notes about what the team learned.

{% assign seasons = site.matches | group_by: "season" | sort: "name" | reverse %}
{% for season in seasons %}
  {% assign events = season.items | sort: "order" %}
  {% assign games = events | map: "game" | uniq | join: ", " %}
  {% assign total_matches = 0 %}
  {% for event in events %}
    {% assign total_matches = total_matches | plus: event.matches_count %}
  {% endfor %}

<section class="season-section" id="season-{{ season.name | slugify }}">
  <div class="season-heading">
    <p class="section-kicker">{{ games }}</p>
    <h2>{{ season.name }}</h2>
    <p>{{ events.size }} events · {{ total_matches }} matches</p>
  </div>

  <div class="listing-grid">
  {% for item in events %}
    <article class="listing-card">
      <p class="card-meta">{{ item.event_date }}{% if item.location %} · {{ item.location }}{% endif %}</p>
      <h3>{{ item.title }}</h3>
      <p>{{ item.summary }}</p>
      <p class="tag-row">
        {% if item.event_type %}<span>{{ item.event_type }}</span>{% endif %}
        {% if item.division %}<span>{{ item.division }} division</span>{% endif %}
        {% if item.record %}<span>{{ item.record }}</span>{% endif %}
        {% if item.matches_count %}<span>{{ item.matches_count }} matches</span>{% endif %}
      </p>
      {% if item.match_sections %}
      <div class="match-sections">
        {% for section in item.match_sections %}
        <section class="match-section">
          <h4>{{ section.title }}</h4>
          <ul class="match-list">
            {% for match in section.matches %}
            <li class="{{ match.result }}">
              <span class="match-result">{% if match.result == "win" %}🏆{% else %}❌{% endif %}</span>
              <a href="{{ match.official_url }}" target="_blank" rel="noopener">{{ match.label }}</a>
              {% if match.video_url %}
              <a class="video-link" href="{{ match.video_url }}" target="_blank" rel="noopener">video</a>
              {% endif %}
            </li>
            {% endfor %}
          </ul>
        </section>
        {% endfor %}
      </div>
      {% endif %}
    </article>
  {% endfor %}
  </div>
</section>
{% endfor %}
