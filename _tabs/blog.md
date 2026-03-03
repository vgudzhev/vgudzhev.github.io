---
layout: page
title: Blog
icon: fas fa-pen-nib
order: 1
---

{% assign postsByYear = site.posts | group_by_exp: "post", "post.date | date: '%Y'" %}
{% for year in postsByYear %}
## {{ year.name }}

{% for post in year.items %}
- [{{ post.title }}]({{ post.url }}) — {{ post.date | date: "%b %d, %Y" }}
{% endfor %}
{% endfor %}
