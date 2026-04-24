---
icon: fas fa-box-open
order: 6
---

{% assign posts = site.posts | where_exp: "post", "post.tags contains 'hackthebox'" %}

{% for post in posts %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%b %d, %Y" }}
{% endfor %}
