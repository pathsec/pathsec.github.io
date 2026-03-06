---
layout: default
title: Archive
permalink: /archive/
---

<div class="container">
  <section class="post-list">
    <p class="section-label">all posts</p>

    {% assign posts_by_year = site.posts | group_by_exp: "post", "post.date | date: '%Y'" %}
    {% for year_group in posts_by_year %}
      <h2 style="font-family: var(--font-mono); font-size: 0.8rem; letter-spacing: 0.1em; color: var(--muted); text-transform: uppercase; padding: 2rem 0 0.5rem; border-top: 1px solid var(--border); margin-top: 1rem;">{{ year_group.name }}</h2>
      {% for post in year_group.items %}
      <a class="post-item" href="{{ post.url | relative_url }}">
        <div>
          <h2 class="post-item-title">{{ post.title }}</h2>
        </div>
        <div class="post-meta">{{ post.date | date: "%b %d" }}</div>
      </a>
      {% endfor %}
    {% endfor %}
  </section>
</div>
