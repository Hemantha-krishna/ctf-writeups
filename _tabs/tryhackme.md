---
layout: page
title: TryHackMe
icon: fas fa-flag
order: 5
permalink: /tryhackme/
---

{% assign posts = site.posts | where_exp: "post", "post.tags contains 'tryhackme'" %}

<div class="post-list">
  {% for post in posts %}
  <article class="post-preview">
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    <p class="post-meta">{{ post.date | date: "%b %d, %Y" }}</p>
    {% if post.description %}<p>{{ post.description }}</p>{% endif %}
  </article>
  {% endfor %}
</div>
