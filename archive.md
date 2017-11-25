---
layout: default
title: Post Archive
---

Archive
-------
{% assign postsByYearMonth = site.posts | group_by_exp:"post", "post.date | date: '%Y %b'"  %}
{% for yearMonth in postsByYearMonth %}
### {{ yearMonth.name }}
{% for post in yearMonth.items %}
   * [{{ post.title }}]({{ post.url }})
{% endfor %}
{% endfor %}
