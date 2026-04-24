---
icon: fas fa-flag
order: 5
---

{% assign posts = site.posts | where_exp: "post", "post.tags contains 'tryhackme'" %}

{% for post in posts %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%b %d, %Y" }}
{% endfor %}
