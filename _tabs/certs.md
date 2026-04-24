---
icon: fas fa-certificate
order: 7
---

{% assign posts = site.posts | where_exp: "post", "post.tags contains 'certification'" %}

{% for post in posts %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%b %d, %Y" }}
{% endfor %}
